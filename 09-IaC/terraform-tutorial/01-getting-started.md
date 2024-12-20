# Getting Started with Terraform
# เริ่มต้นใช้งาน Terraform

## Introduction
## บทนำ
Terraform is an Infrastructure as Code (IaC) tool that allows you to build, change, and version infrastructure safely and efficiently.
Terraform เป็นเครื่องมือในการจัดการโครงสร้างพื้นฐาน (Infrastructure as Code) ที่ช่วยให้คุณสามารถสร้าง เปลี่ยนแปลง และจัดการเวอร์ชันของโครงสร้างพื้นฐานได้อย่างปลอดภัยและมีประสิทธิภาพ

## Prerequisites
## สิ่งที่ต้องเตรียม
- AWS Account (บัญชี AWS)
- AWS CLI installed and configured (ติดตั้งและตั้งค่า AWS CLI)
- Terraform installed (version 1.0 or later) (ติดตั้ง Terraform เวอร์ชัน 1.0 ขึ้นไป)
- Basic understanding of cloud concepts (ความเข้าใจพื้นฐานเกี่ยวกับระบบคลาวด์)

## Installation
## การติดตั้ง
1. Download Terraform from the [official website](https://www.terraform.io/downloads.html)
   ดาวน์โหลด Terraform จาก[เว็บไซต์หลัก](https://www.terraform.io/downloads.html)
2. Add Terraform to your system PATH
   เพิ่ม Terraform ลงใน PATH ของระบบ
3. Verify installation:
   ตรวจสอบการติดตั้ง:
```bash
terraform version
```

## Basic Concepts
## แนวคิดพื้นฐาน
- **Provider**: Plugins that Terraform uses to interact with cloud providers
  **Provider**: ปลั๊กอินที่ Terraform ใช้ในการเชื่อมต่อกับผู้ให้บริการคลาวด์
- **Resource**: Infrastructure components you want to manage
  **Resource**: ส่วนประกอบของโครงสร้างพื้นฐานที่คุณต้องการจัดการ
- **State**: Terraform's record of your managed infrastructure
  **State**: บันทึกสถานะของโครงสร้างพื้นฐานที่ Terraform จัดการ
- **Plan**: Preview of changes Terraform will make
  **Plan**: ดูตัวอย่างการเปลี่ยนแปลงที่ Terraform จะทำ
- **Apply**: Execute the planned changes
  **Apply**: ดำเนินการตามแผนที่วางไว้

## Your First Terraform Configuration
## การตั้งค่า Terraform ครั้งแรกของคุณ
Create a new file named `main.tf`:
สร้างไฟล์ใหม่ชื่อ `main.tf`:

```hcl
# Configure the AWS Provider
# ตั้งค่า AWS Provider
provider "aws" {
  region = "us-west-2"
}

# Create an S3 bucket
# สร้าง S3 bucket
resource "aws_s3_bucket" "example_bucket" {
  bucket = "my-terraform-example-bucket-2024"
  
  tags = {
    Name        = "My Example Bucket"
    Environment = "Dev"
    Managed_by  = "Terraform"
  }
}

# Enable versioning
# เปิดใช้งานการจัดการเวอร์ชัน
resource "aws_s3_bucket_versioning" "versioning_example" {
  bucket = aws_s3_bucket.example_bucket.id
  versioning_configuration {
    status = "Enabled"
  }
}

# Block public access
# ปิดกั้นการเข้าถึงจากภายนอก
resource "aws_s3_bucket_public_access_block" "example" {
  bucket = aws_s3_bucket.example_bucket.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
```

## Workflow Steps
## ขั้นตอนการทำงาน

1. **Initialize Terraform** (เริ่มต้นใช้งาน Terraform)
```bash
terraform init
```
This downloads required provider plugins.
คำสั่งนี้จะดาวน์โหลดปลั๊กอินที่จำเป็น

2. **Format Your Code** (จัดรูปแบบโค้ด)
```bash
terraform fmt
```
This ensures consistent formatting.
คำสั่งนี้ช่วยจัดรูปแบบโค้ดให้สวยงามและเป็นมาตรฐาน

3. **Validate Configuration** (ตรวจสอบการตั้งค่า)
```bash
terraform validate
```
This checks for configuration errors.
ตรวจสอบข้อผิดพลาดในการตั้งค่า

4. **Plan Changes** (วางแผนการเปลี่ยนแปลง)
```bash
terraform plan
```
This shows what changes will be made.
แสดงการเปลี่ยนแปลงที่จะเกิดขึ้น

5. **Apply Changes** (ดำเนินการเปลี่ยนแปลง)
```bash
terraform apply
```
This creates the infrastructure.
สร้างโครงสร้างพื้นฐานตามที่กำหนด

6. **Destroy Resources** (ลบทรัพยากร - เมื่อต้องการ)
```bash
terraform destroy
```
This removes all created resources.
ลบทรัพยากรทั้งหมดที่สร้างไว้

## Best Practices
## แนวทางปฏิบัติที่ดี
1. Always use version control (ใช้ระบบควบคุมเวอร์ชันเสมอ)
2. Use consistent naming conventions (ใช้การตั้งชื่อที่สอดคล้องกัน)
3. Use variables for reusable values (ใช้ตัวแปรสำหรับค่าที่ใช้ซ้ำ)
4. Keep your state file secure (เก็บไฟล์ state ให้ปลอดภัย)
5. Run `terraform plan` before `apply` (รัน `terraform plan` ก่อน `apply` เสมอ)

## Next Steps
## ขั้นตอนต่อไป
- Learn about Terraform variables and outputs (เรียนรู้เกี่ยวกับตัวแปรและผลลัพธ์ใน Terraform)
- Explore more AWS resources (สำรวจทรัพยากร AWS เพิ่มเติม)
- Study Terraform state management (ศึกษาการจัดการ state ใน Terraform)
- Learn about workspaces and environments (เรียนรู้เกี่ยวกับ workspaces และสภาพแวดล้อม)

## Common Commands Reference
## คำสั่งที่ใช้บ่อย
- `terraform init`: Initialize working directory (เริ่มต้นไดเรกทอรีทำงาน)
- `terraform plan`: Preview changes (ดูตัวอย่างการเปลี่ยนแปลง)
- `terraform apply`: Apply changes (ดำเนินการเปลี่ยนแปลง)
- `terraform destroy`: Remove resources (ลบทรัพยากร)
- `terraform fmt`: Format configuration (จัดรูปแบบการตั้งค่า)
- `terraform validate`: Validate configuration (ตรวจสอบการตั้งค่า)
- `terraform show`: Show current state (แสดงสถานะปัจจุบัน)
- `terraform state list`: List resources in state (แสดงรายการทรัพยากรใน state)
