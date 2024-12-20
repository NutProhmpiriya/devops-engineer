# Variables and Outputs in Terraform
# ตัวแปรและการแสดงผลใน Terraform

## Variables (ตัวแปร)
ตัวแปรใน Terraform ช่วยให้เราสามารถกำหนดค่าที่ใช้ซ้ำหรือค่าที่อาจเปลี่ยนแปลงได้ในอนาคต

### Variable Types (ประเภทของตัวแปร)
- string: ข้อความ
- number: ตัวเลข
- bool: ค่าความจริง (true/false)
- list: รายการข้อมูล
- map: ข้อมูลแบบ key-value
- object: โครงสร้างข้อมูลที่ซับซ้อน

### Example Variables File (variables.tf)
```hcl
# Basic string variable
# ตัวแปรพื้นฐานแบบข้อความ
variable "aws_region" {
  description = "AWS region to deploy resources"
  type        = string
  default     = "us-west-2"
}

# List variable
# ตัวแปรแบบรายการ
variable "allowed_ports" {
  description = "List of allowed ports"
  type        = list(number)
  default     = [80, 443, 22]
}

# Map variable
# ตัวแปรแบบ map
variable "tags" {
  description = "Common tags for all resources"
  type        = map(string)
  default     = {
    Environment = "Development"
    Project     = "Tutorial"
  }
}
```

### Using Variables (การใช้งานตัวแปร)
```hcl
provider "aws" {
  region = var.aws_region
}

resource "aws_instance" "example" {
  # ... other configurations ...
  tags = var.tags
}
```

## Outputs (การแสดงผล)
Outputs ใช้สำหรับแสดงค่าที่สำคัญหลังจากสร้าง Infrastructure เสร็จแล้ว

### Example Outputs File (outputs.tf)
```hcl
# Basic output
# การแสดงผลพื้นฐาน
output "instance_ip" {
  description = "Public IP of the instance"
  value       = aws_instance.example.public_ip
}

# Complex output
# การแสดงผลแบบซับซ้อน
output "instance_details" {
  description = "Details about the created instance"
  value = {
    id         = aws_instance.example.id
    public_ip  = aws_instance.example.public_ip
    private_ip = aws_instance.example.private_ip
    tags       = aws_instance.example.tags
  }
}
```

## Variable Files (.tfvars)
`.tfvars` ไฟล์ใช้สำหรับกำหนดค่าตัวแปรแยกตามสภาพแวดล้อม

### Example: development.tfvars
```hcl
aws_region = "us-west-2"
tags = {
  Environment = "Development"
  Project     = "Tutorial"
}
```

### Example: production.tfvars
```hcl
aws_region = "us-east-1"
tags = {
  Environment = "Production"
  Project     = "Tutorial"
}
```

## การใช้งาน Variable Files
```bash
# ใช้งานกับ environment development
terraform plan -var-file="development.tfvars"

# ใช้งานกับ environment production
terraform plan -var-file="production.tfvars"
```

## Best Practices (แนวทางปฏิบัติที่ดี)
1. ตั้งชื่อตัวแปรให้มีความหมายและเข้าใจง่าย
2. เพิ่ม description ให้กับทุกตัวแปรและ output
3. กำหนดค่า default ให้กับตัวแปรที่ไม่จำเป็นต้องระบุทุกครั้ง
4. แยกไฟล์ตัวแปรตามสภาพแวดล้อม (development, staging, production)
5. ใช้ output เพื่อแสดงข้อมูลที่สำคัญหลังจากสร้าง infrastructure
