# Terraform Modules
# โมดูลใน Terraform

## What are Modules?
## โมดูลคืออะไร?

โมดูลคือชุดของ Terraform configuration ที่สามารถนำกลับมาใช้ใหม่ได้ ช่วยให้:
1. ลดการเขียนโค้ดซ้ำ
2. จัดการโค้ดได้ง่ายขึ้น
3. แบ่งปันการทำงานระหว่างทีม

## Module Structure (โครงสร้างโมดูล)
```
modules/
  ├── vpc/
  │   ├── main.tf
  │   ├── variables.tf
  │   ├── outputs.tf
  │   └── README.md
  └── ec2/
      ├── main.tf
      ├── variables.tf
      ├── outputs.tf
      └── README.md
```

## Creating a Module (การสร้างโมดูล)

### Example VPC Module (โมดูล VPC ตัวอย่าง)

#### modules/vpc/variables.tf
```hcl
variable "vpc_cidr" {
  description = "CIDR block for VPC"
  type        = string
  default     = "10.0.0.0/16"
}

variable "environment" {
  description = "Environment name"
  type        = string
}
```

#### modules/vpc/main.tf
```hcl
resource "aws_vpc" "main" {
  cidr_block = var.vpc_cidr
  
  tags = {
    Name        = "main-vpc"
    Environment = var.environment
  }
}

resource "aws_subnet" "public" {
  vpc_id     = aws_vpc.main.id
  cidr_block = cidrsubnet(var.vpc_cidr, 8, 1)
  
  tags = {
    Name        = "public-subnet"
    Environment = var.environment
  }
}
```

#### modules/vpc/outputs.tf
```hcl
output "vpc_id" {
  description = "ID of the created VPC"
  value       = aws_vpc.main.id
}

output "public_subnet_id" {
  description = "ID of the public subnet"
  value       = aws_subnet.public.id
}
```

## Using Modules (การใช้งานโมดูล)

### Local Module (โมดูลแบบ Local)
```hcl
module "vpc" {
  source = "./modules/vpc"
  
  vpc_cidr    = "10.0.0.0/16"
  environment = "production"
}

# Use module outputs
# การใช้งาน output จากโมดูล
resource "aws_instance" "example" {
  subnet_id = module.vpc.public_subnet_id
  # ... other configurations ...
}
```

### Remote Module (โมดูลแบบ Remote)
```hcl
module "vpc" {
  source = "terraform-aws-modules/vpc/aws"
  version = "3.2.0"
  
  name = "my-vpc"
  cidr = "10.0.0.0/16"
  
  azs             = ["us-west-2a", "us-west-2b"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24"]
}
```

## Module Best Practices (แนวทางปฏิบัติที่ดีสำหรับโมดูล)

1. การตั้งชื่อและโครงสร้าง
   - ใช้ชื่อที่สื่อความหมาย
   - จัดโครงสร้างไฟล์ให้เป็นมาตรฐาน
   - เขียน README ให้ละเอียด

2. การจัดการตัวแปร
   - กำหนดค่า default ที่เหมาะสม
   - ใส่ description ให้ครบถ้วน
   - ตรวจสอบ type ของตัวแปร

3. Outputs
   - ส่งออกค่าที่จำเป็นเท่านั้น
   - ตั้งชื่อ output ให้ชัดเจน
   - ใส่ description ทุกครั้ง

4. Versioning
   - ใช้ Semantic Versioning
   - ระบุ version ในการใช้งาน
   - อัพเดท CHANGELOG

## Example Project Structure (โครงสร้างโปรเจคตัวอย่าง)
```
project/
├── main.tf           # Main configuration
├── variables.tf      # Project variables
├── outputs.tf        # Project outputs
├── versions.tf       # Required providers
├── terraform.tfvars  # Variable values
└── modules/
    ├── vpc/         # VPC module
    ├── ec2/         # EC2 module
    └── rds/         # RDS module
```

## Testing Modules (การทดสอบโมดูล)
```hcl
# examples/complete/main.tf
module "test_vpc" {
  source = "../../modules/vpc"
  
  vpc_cidr    = "10.0.0.0/16"
  environment = "test"
}
```

## Documentation (การเขียนเอกสาร)
```markdown
# VPC Module

## Usage
```hcl
module "vpc" {
  source = "./modules/vpc"
  
  vpc_cidr    = "10.0.0.0/16"
  environment = "production"
}
```

## Inputs
| Name | Description | Type | Default |
|------|-------------|------|---------|
| vpc_cidr | CIDR block for VPC | string | "10.0.0.0/16" |
| environment | Environment name | string | n/a |

## Outputs
| Name | Description |
|------|-------------|
| vpc_id | ID of the created VPC |
| public_subnet_id | ID of the public subnet |
```
