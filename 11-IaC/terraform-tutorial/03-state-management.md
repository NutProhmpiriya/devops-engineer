# State Management in Terraform
# การจัดการ State ใน Terraform

## What is Terraform State?
## State คืออะไร?

Terraform State คือไฟล์ที่เก็บข้อมูลเกี่ยวกับ Infrastructure ที่ Terraform จัดการ ซึ่งใช้ในการ:
1. ติดตามทรัพยากรที่สร้างไว้
2. ตรวจสอบการเปลี่ยนแปลง
3. จัดการการทำงานแบบทีม

## Local State vs Remote State
## State แบบ Local และ Remote

### Local State (State แบบ Local)
```hcl
# Default behavior - state stored in terraform.tfstate
# พฤติกรรมเริ่มต้น - state ถูกเก็บในไฟล์ terraform.tfstate
terraform {
  # No backend configuration means local state
  # ไม่มีการตั้งค่า backend หมายถึงใช้ local state
}
```

### Remote State (State แบบ Remote)
```hcl
# AWS S3 as backend
# ใช้ AWS S3 เป็น backend
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-west-2"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}
```

## State Commands (คำสั่งจัดการ State)

### List Resources (แสดงรายการทรัพยากร)
```bash
terraform state list
```

### Show Resource Details (แสดงรายละเอียดทรัพยากร)
```bash
terraform state show aws_s3_bucket.example
```

### Move Resources (ย้ายทรัพยากร)
```bash
terraform state mv aws_s3_bucket.old aws_s3_bucket.new
```

### Remove Resources (ลบทรัพยากร)
```bash
terraform state rm aws_s3_bucket.example
```

## State Locking (การล็อค State)
การล็อค State ป้องกันไม่ให้หลายคนแก้ไข Infrastructure พร้อมกัน

### DynamoDB for State Locking
### ใช้ DynamoDB สำหรับล็อค State
```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-west-2"
    encrypt        = true
    # Use DynamoDB for state locking
    # ใช้ DynamoDB สำหรับล็อค state
    dynamodb_table = "terraform-locks"
  }
}
```

## Workspaces (พื้นที่ทำงาน)
Workspaces ช่วยให้จัดการหลาย environment ในชุด code เดียวกัน

### Workspace Commands (คำสั่งจัดการ Workspace)
```bash
# List workspaces
# แสดงรายการ workspace
terraform workspace list

# Create new workspace
# สร้าง workspace ใหม่
terraform workspace new dev

# Switch workspace
# สลับ workspace
terraform workspace select prod

# Delete workspace
# ลบ workspace
terraform workspace delete dev
```

## Best Practices (แนวทางปฏิบัติที่ดี)
1. ใช้ Remote State เสมอในการทำงานเป็นทีม
2. เข้ารหัสข้อมูลใน State File
3. ใช้ State Locking เพื่อป้องกันการแก้ไขพร้อมกัน
4. แยก State ตาม Environment
5. สำรองข้อมูล State อย่างสม่ำเสมอ
6. ตรวจสอบ State ก่อนทำการเปลี่ยนแปลงสำคัญ
