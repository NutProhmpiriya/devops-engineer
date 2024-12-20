# แบบฝึกหัดที่ 8: การจัดการ IAM และ Security
# Exercise 8: Managing IAM and Security

## โจทย์ | Problem
ตั้งค่าระบบความปลอดภัยและการจัดการสิทธิ์สำหรับทีม DevOps โดยมีความต้องการดังนี้:

1. สร้างและจัดการ:
   - Custom roles สำหรับทีม DevOps
   - Service accounts สำหรับแต่ละ environment
   - Organization policies
   - Security policies

2. ตั้งค่า:
   - Audit logging
   - Cloud KMS
   - Secret Manager
   - VPC Service Controls

## เฉลย | Solution

### 1. สร้าง Custom Roles:
```bash
# Create DevOps role definition
cat > devops-role.yaml << EOF
title: "DevOps Engineer"
description: "Custom role for DevOps team"
stage: "GA"
includedPermissions:
- compute.instances.get
- compute.instances.list
- compute.instances.start
- compute.instances.stop
- container.clusters.get
- container.clusters.list
- container.clusters.update
- storage.buckets.get
- storage.buckets.list
- cloudsql.instances.get
- cloudsql.instances.list
EOF

# Create the role
gcloud iam roles create devops_engineer \
    --project=$PROJECT_ID \
    --file=devops-role.yaml

# Create role for different environments
for env in dev staging prod; do
    gcloud iam roles copy devops_engineer \
        --source-project=$PROJECT_ID \
        --destination-project=$PROJECT_ID \
        --destination-role=devops_engineer_${env}
done
```

### 2. สร้าง Service Accounts:
```bash
# Create service accounts for each environment
for env in dev staging prod; do
    # Create service account
    gcloud iam service-accounts create sa-${env} \
        --display-name="Service Account for ${env}"
    
    # Assign role
    gcloud projects add-iam-policy-binding $PROJECT_ID \
        --member="serviceAccount:sa-${env}@$PROJECT_ID.iam.gserviceaccount.com" \
        --role="projects/$PROJECT_ID/roles/devops_engineer_${env}"
done
```

### 3. ตั้งค่า Organization Policies:
```bash
# Set organization policies
gcloud resource-manager org-policies set-policy \
    --project=$PROJECT_ID policy.yaml

cat > policy.yaml << EOF
constraint: compute.vmExternalIpAccess
listPolicy:
  allowedValues:
  - "projects/$PROJECT_ID/zones/asia-southeast1-a"
EOF

# Disable service account key creation
gcloud resource-manager org-policies set-policy \
    --project=$PROJECT_ID << EOF
constraint: iam.disableServiceAccountKeyCreation
booleanPolicy:
  enforced: true
EOF
```

### 4. ตั้งค่า Cloud KMS:
```bash
# Create keyring
gcloud kms keyrings create app-keyring \
    --location=global

# Create encryption key
gcloud kms keys create app-key \
    --location=global \
    --keyring=app-keyring \
    --purpose=encryption

# Grant access to service accounts
for env in dev staging prod; do
    gcloud kms keys add-iam-policy-binding app-key \
        --location=global \
        --keyring=app-keyring \
        --member="serviceAccount:sa-${env}@$PROJECT_ID.iam.gserviceaccount.com" \
        --role=roles/cloudkms.cryptoKeyEncrypterDecrypter
done
```

### 5. ตั้งค่า Secret Manager:
```bash
# Create secrets for each environment
for env in dev staging prod; do
    # Create database credentials
    echo "db-password-${env}" | \
    gcloud secrets create db-password-${env} \
        --data-file=- \
        --replication-policy="automatic"
    
    # Grant access
    gcloud secrets add-iam-policy-binding db-password-${env} \
        --member="serviceAccount:sa-${env}@$PROJECT_ID.iam.gserviceaccount.com" \
        --role="roles/secretmanager.secretAccessor"
done
```

### 6. ตั้งค่า VPC Service Controls:
```bash
# Create service perimeter
gcloud access-context-manager perimeters create app-perimeter \
    --title="Application Perimeter" \
    --resources="projects/$PROJECT_ID" \
    --restricted-services="storage.googleapis.com,container.googleapis.com" \
    --access-levels="accessPolicies/$POLICY_ID/accessLevels/trusted_networks"

# Create access level for trusted networks
cat > trusted-networks.yaml << EOF
name: accessPolicies/$POLICY_ID/accessLevels/trusted_networks
title: "Trusted Networks"
basic:
  conditions:
  - ipSubnetworks:
    - "10.0.0.0/8"
    - "172.16.0.0/12"
EOF

gcloud access-context-manager levels create trusted_networks \
    --policy=$POLICY_ID \
    --file=trusted-networks.yaml
```

### 7. ตั้งค่า Audit Logging:
```bash
# Enable audit logging
gcloud projects get-iam-policy $PROJECT_ID > policy.yaml

# Add audit config
cat >> policy.yaml << EOF
auditConfigs:
- auditLogConfigs:
  - logType: DATA_READ
  - logType: DATA_WRITE
  - logType: ADMIN_READ
  service: allServices
EOF

gcloud projects set-iam-policy $PROJECT_ID policy.yaml

# Export audit logs to Cloud Storage
gcloud logging sinks create audit-logs \
    storage.googleapis.com/$PROJECT_ID-audit-logs \
    --log-filter='resource.type="project" AND severity>=WARNING'
```

## การทดสอบ | Testing
```bash
# Test IAM roles
gcloud iam roles describe projects/$PROJECT_ID/roles/devops_engineer

# Test service account access
gcloud auth activate-service-account \
    sa-dev@$PROJECT_ID.iam.gserviceaccount.com \
    --key-file=key.json

# Test KMS
echo "test data" | \
gcloud kms encrypt \
    --location=global \
    --keyring=app-keyring \
    --key=app-key \
    --plaintext-file=- \
    --ciphertext-file=encrypted.bin

# Test Secret Manager
gcloud secrets versions access latest \
    --secret=db-password-dev

# View audit logs
gcloud logging read 'resource.type="project"' \
    --limit=5
```

## เพิ่มเติม | Additional Notes
- ใช้ Workload Identity สำหรับ GKE
- ตั้งค่า regular key rotation
- ใช้ Cloud Asset Inventory สำหรับ resource tracking
- พิจารณาใช้ Binary Authorization
- ทำ regular security audits
