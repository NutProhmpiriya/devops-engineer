# Testing in Terraform
# การทดสอบใน Terraform

## Types of Testing (ประเภทของการทดสอบ)

1. Static Analysis (การวิเคราะห์แบบ Static)
   - Syntax validation
   - Format checking
   - Security scanning

2. Unit Testing (การทดสอบระดับ Unit)
   - Module testing
   - Resource configuration testing
   - Variable validation

3. Integration Testing (การทดสอบแบบบูรณาการ)
   - Multi-module testing
   - Cross-resource testing
   - End-to-end testing

## Tools and Frameworks (เครื่องมือและเฟรมเวิร์ค)

### Terratest
```go
package test

import (
    "testing"
    "github.com/gruntwork-io/terratest/modules/terraform"
    "github.com/stretchr/testify/assert"
)

func TestTerraformAwsExample(t *testing.T) {
    terraformOptions := &terraform.Options{
        TerraformDir: "../examples/aws",
        Vars: map[string]interface{}{
            "region": "us-west-2",
            "instance_type": "t2.micro",
        },
    }

    defer terraform.Destroy(t, terraformOptions)
    terraform.InitAndApply(t, terraformOptions)

    instanceID := terraform.Output(t, terraformOptions, "instance_id")
    assert.NotEmpty(t, instanceID)
}
```

### Kitchen-Terraform
```ruby
# .kitchen.yml
driver:
  name: terraform
  root_module_directory: test/fixtures/my_module

provisioner:
  name: terraform

platforms:
  - name: aws

verifier:
  name: terraform
  systems:
    - name: default
      backend: ssh
      hosts_output: public_ip
      user: ubuntu
      key_files:
        - test/assets/keypair.pem

suites:
  - name: default
```

### Terraform Built-in Testing
```hcl
# Variable validation
variable "instance_type" {
  type = string
  validation {
    condition     = can(regex("^t2\\.", var.instance_type))
    error_message = "Instance type must be t2 series."
  }
}

# Custom conditions
check "instance_count" {
  assert {
    condition     = var.instance_count > 0 && var.instance_count <= 10
    error_message = "Instance count must be between 1 and 10."
  }
}
```

## Test Examples (ตัวอย่างการทดสอบ)

### 1. Module Testing
```hcl
# modules/vpc/test/main.tf
module "test_vpc" {
  source = "../"
  
  vpc_cidr = "10.0.0.0/16"
  environment = "test"
}

# Test outputs
output "vpc_id" {
  value = module.test_vpc.vpc_id
}
```

### 2. Resource Testing
```hcl
# Test AWS S3 bucket configuration
resource "aws_s3_bucket" "test" {
  bucket = "test-bucket"
  
  lifecycle {
    prevent_destroy = false
  }
}

resource "aws_s3_bucket_versioning" "test" {
  bucket = aws_s3_bucket.test.id
  versioning_configuration {
    status = "Enabled"
  }
}
```

### 3. Integration Testing
```go
func TestFullStackDeployment(t *testing.T) {
    terraformOptions := &terraform.Options{
        TerraformDir: "../",
        Vars: map[string]interface{}{
            "environment": "test",
        },
    }

    defer terraform.Destroy(t, terraformOptions)

    // Deploy infrastructure
    terraform.InitAndApply(t, terraformOptions)

    // Test VPC
    vpcId := terraform.Output(t, terraformOptions, "vpc_id")
    assert.NotEmpty(t, vpcId)

    // Test EC2
    instanceId := terraform.Output(t, terraformOptions, "instance_id")
    assert.NotEmpty(t, instanceId)

    // Test connectivity
    publicIp := terraform.Output(t, terraformOptions, "public_ip")
    testHttpConnection(t, publicIp)
}
```

## Best Practices (แนวทางปฏิบัติที่ดี)

1. การเขียน Test
   - แยก test files ให้ชัดเจน
   - ทำ test coverage ให้ครอบคลุม
   - เขียน test cases ที่มีความหมาย

2. Test Environment
   - แยก test environment
   - ใช้ temporary resources
   - ทำ cleanup หลัง test

3. Continuous Testing
   - รวม tests ใน CI/CD
   - ทำ automated testing
   - ตรวจสอบผลทดสอบสม่ำเสมอ

4. Documentation
   - อธิบาย test cases
   - ทำ test documentation
   - บันทึกผลการทดสอบ

## Testing Workflow (ขั้นตอนการทดสอบ)

1. Pre-commit
   - ตรวจสอบ syntax
   - รัน format check
   - ทำ static analysis

2. CI Pipeline
   - รัน unit tests
   - ทำ security scan
   - ตรวจสอบ compliance

3. Integration Testing
   - ทดสอบ modules รวมกัน
   - ตรวจสอบการทำงานร่วมกัน
   - ทดสอบ end-to-end

4. Performance Testing
   - ทดสอบ scalability
   - วัด response time
   - ตรวจสอบ resource usage

## Security Testing (การทดสอบความปลอดภัย)

1. Static Analysis
   - ใช้ security scanners
   - ตรวจสอบ best practices
   - หา vulnerabilities

2. Compliance Testing
   - ตรวจสอบ compliance rules
   - ทำ policy validation
   - รัน security benchmarks

3. Access Control Testing
   - ทดสอบ IAM policies
   - ตรวจสอบ permissions
   - ทดสอบ authentication
