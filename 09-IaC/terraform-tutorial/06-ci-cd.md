# CI/CD with Terraform
# การทำ CI/CD กับ Terraform

## Overview (ภาพรวม)
การทำ CI/CD กับ Terraform ช่วยให้:
- ทดสอบการเปลี่ยนแปลงอัตโนมัติ
- ลดความผิดพลาดจากมนุษย์
- เพิ่มความเร็วในการ deploy
- ติดตามการเปลี่ยนแปลงได้ดีขึ้น

## GitLab CI/CD Example
### .gitlab-ci.yml
```yaml
image: hashicorp/terraform:latest

variables:
  TF_ROOT: ${CI_PROJECT_DIR}/terraform
  TF_STATE_NAME: ${CI_PROJECT_PATH_SLUG}

cache:
  paths:
    - ${TF_ROOT}/.terraform

before_script:
  - cd ${TF_ROOT}
  - terraform init

stages:
  - validate
  - plan
  - apply

validate:
  stage: validate
  script:
    - terraform validate

plan:
  stage: plan
  script:
    - terraform plan -out=plan.tfplan
  artifacts:
    paths:
      - plan.tfplan

apply:
  stage: apply
  script:
    - terraform apply -auto-approve plan.tfplan
  dependencies:
    - plan
  only:
    - master
```

## GitHub Actions Example
### .github/workflows/terraform.yml
```yaml
name: 'Terraform CI/CD'

on:
  push:
    branches:
    - main
  pull_request:

jobs:
  terraform:
    name: 'Terraform'
    runs-on: ubuntu-latest

    steps:
    - name: Checkout
      uses: actions/checkout@v2

    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v1

    - name: Terraform Init
      run: terraform init

    - name: Terraform Format
      run: terraform fmt -check

    - name: Terraform Plan
      run: terraform plan -no-color
      if: github.event_name == 'pull_request'

    - name: Terraform Apply
      if: github.ref == 'refs/heads/main' && github.event_name == 'push'
      run: terraform apply -auto-approve
```

## Jenkins Pipeline Example
### Jenkinsfile
```groovy
pipeline {
    agent any
    
    environment {
        TF_HOME = tool('terraform')
        PATH = "${TF_HOME}:${PATH}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Terraform Init') {
            steps {
                sh 'terraform init'
            }
        }
        
        stage('Terraform Plan') {
            steps {
                sh 'terraform plan -out=tfplan'
                sh 'terraform show -no-color tfplan > tfplan.txt'
            }
        }
        
        stage('Approval') {
            when {
                branch 'main'
            }
            steps {
                input 'Apply the terraform plan?'
            }
        }
        
        stage('Terraform Apply') {
            when {
                branch 'main'
            }
            steps {
                sh 'terraform apply -auto-approve tfplan'
            }
        }
    }
}
```

## Testing in CI/CD (การทดสอบใน CI/CD)
```hcl
# Example test using Terratest
package test

import (
    "testing"
    "github.com/gruntwork-io/terratest/modules/terraform"
    "github.com/stretchr/testify/assert"
)

func TestTerraformBasicExample(t *testing.T) {
    terraformOptions := &terraform.Options{
        TerraformDir: "../examples/basic",
    }

    defer terraform.Destroy(t, terraformOptions)
    terraform.InitAndApply(t, terraformOptions)

    output := terraform.Output(t, terraformOptions, "instance_ip")
    assert.NotEmpty(t, output)
}
```

## Security Considerations (ข้อควรพิจารณาด้านความปลอดภัย)

1. Credentials Management
   - ใช้ secrets management
   - หลีกเลี่ยงการเก็บ credentials ใน code
   - ใช้ temporary credentials

2. Access Control
   - จำกัดสิทธิ์การ apply
   - แยก environments
   - ใช้ approval process

3. Audit Trail
   - เก็บ logs ทั้งหมด
   - ติดตามการเปลี่ยนแปลง
   - ทำ compliance reporting

## Best Practices (แนวทางปฏิบัติที่ดี)

1. Workflow
   - แยก environments ชัดเจน
   - ใช้ branch protection
   - มี approval process

2. Testing
   - ทำ automated testing
   - ทดสอบก่อน apply
   - มี rollback plan

3. Documentation
   - อธิบายการเปลี่ยนแปลง
   - เก็บประวัติการ deploy
   - ทำ runbook

4. Monitoring
   - ติดตามการ deploy
   - แจ้งเตือนเมื่อมีปัญหา
   - วิเคราะห์ประสิทธิภาพ
