# แบบฝึกหัดที่ 4: การตั้งค่า CI/CD ด้วย Cloud Build
# Exercise 4: Setting up CI/CD with Cloud Build

## โจทย์ | Problem
สร้าง CI/CD pipeline ด้วย Cloud Build สำหรับ Node.js application โดยมีความต้องการดังนี้:

1. สร้าง pipeline ที่ประกอบด้วย:
   - Unit testing
   - Building Docker image
   - Vulnerability scanning
   - Pushing to Container Registry
   - Deploying to Cloud Run

2. ตั้งค่า triggers สำหรับ:
   - Push to main branch
   - Pull request to main branch
   - Tag creation (release)

3. ตั้งค่า notifications เมื่อ build สำเร็จหรือล้มเหลว

## เฉลย | Solution

### 1. สร้าง Cloud Build Configuration (cloudbuild.yaml):
```yaml
steps:
# Install dependencies and run tests
- name: 'node:16'
  entrypoint: npm
  args: ['install']
  
- name: 'node:16'
  entrypoint: npm
  args: ['test']

# Build Docker image
- name: 'gcr.io/cloud-builders/docker'
  args: [
    'build',
    '-t', 'gcr.io/$PROJECT_ID/myapp:$COMMIT_SHA',
    '.'
  ]

# Vulnerability scanning
- name: 'gcr.io/cloud-builders/gcloud'
  args: [
    'container', 'images',
    'scan',
    'gcr.io/$PROJECT_ID/myapp:$COMMIT_SHA',
    '--format=json'
  ]
  id: 'Scan Container'

# Push container image
- name: 'gcr.io/cloud-builders/docker'
  args: [
    'push',
    'gcr.io/$PROJECT_ID/myapp:$COMMIT_SHA'
  ]

# Deploy to Cloud Run
- name: 'gcr.io/cloud-builders/gcloud'
  args: [
    'run', 'deploy', 'myapp',
    '--image', 'gcr.io/$PROJECT_ID/myapp:$COMMIT_SHA',
    '--region', 'asia-southeast1',
    '--platform', 'managed',
    '--allow-unauthenticated'
  ]

# Tag latest image
- name: 'gcr.io/cloud-builders/docker'
  args: [
    'tag',
    'gcr.io/$PROJECT_ID/myapp:$COMMIT_SHA',
    'gcr.io/$PROJECT_ID/myapp:latest'
  ]

images:
- 'gcr.io/$PROJECT_ID/myapp:$COMMIT_SHA'
- 'gcr.io/$PROJECT_ID/myapp:latest'
```

### 2. สร้าง Dockerfile:
```dockerfile
FROM node:16-alpine

WORKDIR /app
COPY package*.json ./
RUN npm install --production
COPY . .

EXPOSE 3000
CMD ["npm", "start"]
```

### 3. ตั้งค่า Triggers ด้วย gcloud:
```bash
# Create trigger for main branch
gcloud builds triggers create github \
  --name="main-branch-trigger" \
  --repo-name="myapp" \
  --repo-owner="$GITHUB_USER" \
  --branch-pattern="^main$" \
  --build-config="cloudbuild.yaml"

# Create trigger for pull requests
gcloud builds triggers create github \
  --name="pr-trigger" \
  --repo-name="myapp" \
  --repo-owner="$GITHUB_USER" \
  --pull-request-pattern="^main$" \
  --build-config="cloudbuild.yaml"

# Create trigger for tags
gcloud builds triggers create github \
  --name="tag-trigger" \
  --repo-name="myapp" \
  --repo-owner="$GITHUB_USER" \
  --tag-pattern="v.*" \
  --build-config="cloudbuild.yaml"
```

### 4. ตั้งค่า Notifications:
```bash
# Create Pub/Sub topic
gcloud pubsub topics create cloud-builds

# Create Cloud Function for notifications
cat > index.js << EOF
const {Storage} = require('@google-cloud/storage');
const storage = new Storage();

exports.notifyBuildStatus = async (event, context) => {
  const build = JSON.parse(Buffer.from(event.data, 'base64').toString());
  
  if (build.status === 'SUCCESS' || build.status === 'FAILURE') {
    // Send notification (example using Slack webhook)
    const message = {
      text: `Build ${build.id} ${build.status}\nTrigger: ${build.trigger.name}\nRepo: ${build.source.repoSource.repoName}`
    };
    
    await fetch(process.env.SLACK_WEBHOOK_URL, {
      method: 'POST',
      headers: {'Content-Type': 'application/json'},
      body: JSON.stringify(message)
    });
  }
};
EOF

# Deploy Cloud Function
gcloud functions deploy notifyBuildStatus \
  --runtime nodejs16 \
  --trigger-topic cloud-builds \
  --set-env-vars SLACK_WEBHOOK_URL=YOUR_SLACK_WEBHOOK_URL
```

## การทดสอบ | Testing
```bash
# Trigger manual build
gcloud builds submit --config=cloudbuild.yaml .

# Check build status
gcloud builds list

# View build logs
gcloud builds log $(gcloud builds list --limit=1 --format='value(id)')

# Test Cloud Run deployment
gcloud run services describe myapp --region=asia-southeast1
```

## เพิ่มเติม | Additional Notes
- ใช้ Cloud Build's built-in cache เพื่อเพิ่มความเร็วในการ build
- เพิ่ม security scanning steps ใน pipeline
- ใช้ Artifact Registry แทน Container Registry สำหรับโปรเจคใหม่
- ตั้งค่า timeout และ machine type ตามความเหมาะสม
- พิจารณาใช้ Cloud Build private pools สำหรับความปลอดภัยเพิ่มเติม
