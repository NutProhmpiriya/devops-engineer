# แบบฝึกหัดที่ 18: การใช้งาน Cloud Deploy
# Exercise 18: Using Cloud Deploy

## โจทย์ | Problem
ตั้งค่าและจัดการ continuous delivery pipeline ด้วย Cloud Deploy โดยมีความต้องการดังนี้:

1. สร้าง:
   - Delivery pipeline
   - Targets
   - Release
   - Rollout

2. จัดการ:
   - Approvals
   - Rollbacks
   - Monitoring
   - Security

## เฉลย | Solution

### 1. สร้าง Pipeline Configuration (clouddeploy.yaml):
```yaml
apiVersion: deploy.cloud.google.com/v1
kind: DeliveryPipeline
metadata:
  name: app-pipeline
description: "Main application delivery pipeline"
serialPipeline:
  stages:
  - targetId: dev
    profiles: [dev]
  - targetId: staging
    profiles: [staging]
    strategy:
      canary:
        runtimeConfig:
          cloudRun:
            automaticTrafficControl: true
        steps:
        - percentage: 25
          verify:
            testJob:
              container: gcr.io/project-id/test-runner:latest
  - targetId: prod
    profiles: [prod]
    strategy:
      canary:
        runtimeConfig:
          cloudRun:
            automaticTrafficControl: true
        steps:
        - percentage: 5
        - percentage: 20
        - percentage: 50
        - percentage: 100
```

### 2. สร้าง Target Definitions (targets.yaml):
```yaml
apiVersion: deploy.cloud.google.com/v1
kind: Target
metadata:
  name: dev
description: "Development environment"
requireApproval: false
executionConfigs:
- usages:
  - RENDER
  - DEPLOY
  serviceAccount: deploy-sa@project-id.iam.gserviceaccount.com
  workerPool: projects/project-id/locations/asia-southeast1/workerPools/pool-1
gke:
  cluster: projects/project-id/locations/asia-southeast1/clusters/dev-cluster

---
apiVersion: deploy.cloud.google.com/v1
kind: Target
metadata:
  name: staging
description: "Staging environment"
requireApproval: true
executionConfigs:
- usages:
  - RENDER
  - DEPLOY
  serviceAccount: deploy-sa@project-id.iam.gserviceaccount.com
gke:
  cluster: projects/project-id/locations/asia-southeast1/clusters/staging-cluster

---
apiVersion: deploy.cloud.google.com/v1
kind: Target
metadata:
  name: prod
description: "Production environment"
requireApproval: true
executionConfigs:
- usages:
  - RENDER
  - DEPLOY
  serviceAccount: deploy-sa@project-id.iam.gserviceaccount.com
gke:
  cluster: projects/project-id/locations/asia-southeast1/clusters/prod-cluster
```

### 3. สร้าง Skaffold Configuration (skaffold.yaml):
```yaml
apiVersion: skaffold/v2beta26
kind: Config
metadata:
  name: app
build:
  artifacts:
  - image: gcr.io/project-id/app
    docker:
      dockerfile: Dockerfile
deploy:
  kubectl:
    manifests:
    - k8s/*.yaml
profiles:
- name: dev
  deploy:
    kubectl:
      manifests:
      - k8s/dev/*.yaml
- name: staging
  deploy:
    kubectl:
      manifests:
      - k8s/staging/*.yaml
- name: prod
  deploy:
    kubectl:
      manifests:
      - k8s/prod/*.yaml
```

### 4. สร้าง Cloud Build Configuration (cloudbuild.yaml):
```yaml
steps:
- name: 'gcr.io/cloud-builders/docker'
  args: ['build', '-t', 'gcr.io/$PROJECT_ID/app:$COMMIT_SHA', '.']

- name: 'gcr.io/cloud-builders/docker'
  args: ['push', 'gcr.io/$PROJECT_ID/app:$COMMIT_SHA']

- name: 'gcr.io/google.com/cloudsdktool/cloud-sdk'
  entrypoint: 'gcloud'
  args:
  - 'deploy'
  - 'releases'
  - 'create'
  - 'rel-${SHORT_SHA}'
  - '--delivery-pipeline=app-pipeline'
  - '--region=asia-southeast1'
  - '--annotations=commit_sha=${COMMIT_SHA}'
  - '--source=.'

images:
- 'gcr.io/$PROJECT_ID/app:$COMMIT_SHA'
```

### 5. ตั้งค่า Automation Scripts (deploy_scripts.sh):
```bash
#!/bin/bash

# Create delivery pipeline
gcloud deploy apply --file=clouddeploy.yaml --region=asia-southeast1

# Create targets
gcloud deploy apply --file=targets.yaml --region=asia-southeast1

# Create release
gcloud deploy releases create release-1 \
    --delivery-pipeline=app-pipeline \
    --region=asia-southeast1 \
    --source=. \
    --images=app=gcr.io/project-id/app:latest

# Get release status
gcloud deploy releases list \
    --delivery-pipeline=app-pipeline \
    --region=asia-southeast1

# Promote to next target
gcloud deploy releases promote \
    --release=release-1 \
    --delivery-pipeline=app-pipeline \
    --region=asia-southeast1

# Rollback
gcloud deploy rollouts rollback rollout-id \
    --delivery-pipeline=app-pipeline \
    --region=asia-southeast1
```

### 6. ตั้งค่า Monitoring:
```bash
# Create monitoring dashboard
cat > deploy_dashboard.json << EOF
{
  "displayName": "Cloud Deploy Monitoring",
  "gridLayout": {
    "widgets": [
      {
        "title": "Deployment Success Rate",
        "xyChart": {
          "dataSets": [{
            "timeSeriesQuery": {
              "timeSeriesFilter": {
                "filter": "metric.type=\"clouddeploy.googleapis.com/pipeline/success_count\""
              }
            }
          }]
        }
      },
      {
        "title": "Rollout Duration",
        "xyChart": {
          "dataSets": [{
            "timeSeriesQuery": {
              "timeSeriesFilter": {
                "filter": "metric.type=\"clouddeploy.googleapis.com/pipeline/rollout_duration\""
              }
            }
          }]
        }
      }
    ]
  }
}
EOF

gcloud monitoring dashboards create --config-from-file=deploy_dashboard.json

# Create alerts
gcloud alpha monitoring policies create \
    --display-name="Deployment Failure Alert" \
    --condition-display-name="Deployment failure" \
    --condition-filter='metric.type="clouddeploy.googleapis.com/pipeline/failure_count"' \
    --condition-threshold-value=1 \
    --condition-threshold-duration=0s
```

### 7. ตั้งค่า Security:
```bash
# Create service account
gcloud iam service-accounts create deploy-sa \
    --display-name="Cloud Deploy Service Account"

# Grant permissions
gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member="serviceAccount:deploy-sa@$PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/clouddeploy.operator"

gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member="serviceAccount:deploy-sa@$PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/container.developer"

# Create custom role for approvers
gcloud iam roles create deployApprover \
    --project=$PROJECT_ID \
    --title="Deployment Approver" \
    --description="Role for approving deployments" \
    --permissions="clouddeploy.rollouts.approve"

# Grant approver role
gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member="group:deployers@example.com" \
    --role="projects/$PROJECT_ID/roles/deployApprover"
```

## การทดสอบ | Testing
```bash
# Test pipeline creation
gcloud deploy delivery-pipelines describe app-pipeline \
    --region=asia-southeast1

# Test target creation
gcloud deploy targets list \
    --region=asia-southeast1

# Create test release
gcloud deploy releases create test-release \
    --delivery-pipeline=app-pipeline \
    --region=asia-southeast1 \
    --source=. \
    --annotations=commit_sha=$(git rev-parse HEAD)

# Monitor rollout
gcloud deploy rollouts list \
    --delivery-pipeline=app-pipeline \
    --release=test-release \
    --region=asia-southeast1

# Test rollback
gcloud deploy rollouts rollback ROLLOUT_ID \
    --delivery-pipeline=app-pipeline \
    --region=asia-southeast1
```

## เพิ่มเติม | Additional Notes
- ใช้ Cloud Build triggers สำหรับ automated releases
- ตั้งค่า approval policies ตาม environment
- ใช้ custom metrics สำหรับ canary analysis
- ทำ integration testing ก่อน promotion
- พิจารณาใช้ Cloud Deploy API สำหรับ automation
