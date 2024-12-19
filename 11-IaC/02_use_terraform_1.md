**ตัวอย่างการใช้งาน Terraform** สามารถแสดงให้เห็นกระบวนการสร้าง Infrastructure ตั้งแต่เริ่มต้นจนเสร็จสมบูรณ์ได้ดังนี้:

---

### 1. **ติดตั้ง Terraform**
ดาวน์โหลดและติดตั้ง Terraform จาก [เว็บไซต์ Terraform](https://www.terraform.io/downloads) หรือใช้คำสั่งติดตั้งบนระบบปฏิบัติการที่ใช้งาน เช่น:

#### บน Ubuntu/Debian:
```bash
sudo apt-get update && sudo apt-get install -y terraform
```

#### บน macOS (Homebrew):
```bash
brew install terraform
```

---

### 2. **สร้างไฟล์ Configuration**
สร้างโครงสร้างพื้นฐาน เช่น การสร้าง EC2 Instance บน AWS

#### สร้างไฟล์ `main.tf`:
```hcl
# กำหนด AWS Provider
provider "aws" {
  region = "us-east-1"
}

# สร้าง EC2 Instance
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"  # Amazon Machine Image (AMI)
  instance_type = "t2.micro"               # ประเภทของ Instance

  tags = {
    Name = "MyFirstTerraformInstance"
  }
}
```

---

### 3. **รันคำสั่ง Terraform**
#### ขั้นตอนการทำงาน:
1. **เริ่มต้นโปรเจกต์ (`terraform init`)**:
   ```bash
   terraform init
   ```
   - ดาวน์โหลด Plugin สำหรับ Provider (เช่น AWS)

2. **ตรวจสอบแผนการทำงาน (`terraform plan`)**:
   ```bash
   terraform plan
   ```
   - แสดงรายการทรัพยากรที่จะถูกสร้าง

3. **สร้าง Infrastructure (`terraform apply`)**:
   ```bash
   terraform apply
   ```
   - ใช้คำสั่งนี้เพื่อสร้างทรัพยากรจริง หลังจากที่ยืนยัน (พิมพ์ `yes`)

4. **ลบ Infrastructure (`terraform destroy`)**:
   ```bash
   terraform destroy
   ```
   - ลบทรัพยากรทั้งหมดที่สร้างขึ้นโดย Terraform

---

### 4. **เพิ่มความซับซ้อนด้วย Variables**
สามารถใช้ Variables เพื่อเพิ่มความยืดหยุ่น เช่น การกำหนดค่า Region, AMI, หรือ Instance Type

#### สร้างไฟล์ `variables.tf`:
```hcl
variable "aws_region" {
  default = "us-east-1"
}

variable "instance_type" {
  default = "t2.micro"
}

variable "ami_id" {
  default = "ami-0c55b159cbfafe1f0"
}
```

#### ปรับ `main.tf` ให้ใช้ Variables:
```hcl
provider "aws" {
  region = var.aws_region
}

resource "aws_instance" "example" {
  ami           = var.ami_id
  instance_type = var.instance_type

  tags = {
    Name = "MyTerraformInstanceWithVariables"
  }
}
```

#### รันคำสั่ง:
```bash
terraform apply
```

---

### 5. **ใช้ Modules เพื่อจัดระเบียบโค้ด**
โมดูลช่วยจัดกลุ่มโค้ดให้สามารถนำไปใช้ซ้ำได้

#### ตัวอย่างโมดูล: สร้าง VPC
- สร้างโฟลเดอร์ `modules/vpc`
- ในโฟลเดอร์ `modules/vpc` ให้มีไฟล์:
  - `main.tf`: สำหรับโค้ดหลัก
  - `variables.tf`: สำหรับตัวแปร
  - `outputs.tf`: สำหรับผลลัพธ์

#### ใช้ Module ในโปรเจกต์หลัก:
```hcl
module "vpc" {
  source = "./modules/vpc"

  vpc_name = "MyVPC"
  cidr_block = "10.0.0.0/16"
}
```

---

### 6. **จัดการ State File**
เพื่อหลีกเลี่ยงปัญหาเมื่อทำงานร่วมกันในทีม:
- ใช้ Remote State เช่น S3 + DynamoDB Lock
```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "state/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
  }
}
```

---

### 7. **บูรณาการ Terraform กับ CI/CD**
สามารถรวม Terraform กับ Jenkins, GitLab CI/CD, หรือ GitHub Actions เพื่อทำให้กระบวนการ Deploy โครงสร้างพื้นฐานอัตโนมัติได้ เช่น:
- ใช้ Terraform Plan ในขั้นตอน Preview
- ใช้ Terraform Apply ในขั้นตอน Deploy

