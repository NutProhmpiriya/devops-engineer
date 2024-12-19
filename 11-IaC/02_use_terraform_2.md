Terraform เป็นเครื่องมือ Infrastructure as Code (IaC) ที่ใช้ในการบริหารและจัดการโครงสร้างพื้นฐานผ่านการเขียนโค้ด แทนการใช้คอนโซลหรือ GUI ในการตั้งค่า เราจะมาเรียนรู้พื้นฐานการใช้งาน Terraform กัน

---

### ขั้นตอนการเริ่มต้นใช้งาน Terraform

#### 1. **ติดตั้ง Terraform**
   - ดาวน์โหลด Terraform ได้จาก [terraform.io](https://www.terraform.io/downloads)
   - ติดตั้งในระบบปฏิบัติการที่ใช้งาน เช่น macOS, Linux, หรือ Windows
   - ตรวจสอบการติดตั้งโดยรันคำสั่ง:
     ```bash
     terraform version
     ```

#### 2. **สร้างไฟล์ Configuration**
   - ไฟล์ของ Terraform มักใช้ `.tf` เป็นนามสกุล เช่น `main.tf`
   - โครงสร้างไฟล์พื้นฐาน:
     ```hcl
     provider "aws" {
       region = "us-east-1"
     }

     resource "aws_instance" "example" {
       ami           = "ami-0c55b159cbfafe1f0"
       instance_type = "t2.micro"
     }
     ```

#### 3. **คำสั่งพื้นฐานของ Terraform**
   - **`terraform init`**
     - ใช้สำหรับเตรียมการทำงานของ Terraform เช่น การดาวน์โหลด Plugin
   - **`terraform plan`**
     - แสดงแผนการทำงาน เช่น จะสร้างหรือแก้ไขอะไรบ้าง
   - **`terraform apply`**
     - ใช้สร้างหรือปรับปรุงโครงสร้างพื้นฐานจริง
   - **`terraform destroy`**
     - ลบทุกอย่างที่ถูกสร้างด้วย Terraform

#### 4. **Provider**
   - Terraform รองรับหลาย Provider เช่น AWS, GCP, Azure, DigitalOcean
   - ตัวอย่างการตั้งค่า Provider:
     ```hcl
     provider "aws" {
       region = "us-west-2"
       access_key = "your-access-key"
       secret_key = "your-secret-key"
     }
     ```

#### 5. **State File**
   - Terraform ใช้ไฟล์ `.tfstate` เก็บสถานะของ Infrastructure ที่ถูกสร้าง
   - ห้ามแก้ไขไฟล์นี้โดยตรง

#### 6. **โมดูล (Module)**
   - ใช้สำหรับจัดระเบียบและแบ่งส่วนโค้ดให้ชัดเจน
   - ตัวอย่างโครงสร้างโมดูล:
     ```
     ├── main.tf
     ├── modules
     │   └── instance
     │       ├── main.tf
     │       └── variables.tf
     ```

#### 7. **Variables**
   - ใช้ปรับค่าต่าง ๆ ในโค้ดให้ยืดหยุ่น
   - ตัวอย่างการใช้:
     ```hcl
     variable "instance_type" {
       default = "t2.micro"
     }

     resource "aws_instance" "example" {
       ami           = "ami-0c55b159cbfafe1f0"
       instance_type = var.instance_type
     }
     ```

#### 8. **Output**
   - ใช้แสดงค่าผลลัพธ์หลังจากสร้าง Infrastructure
   - ตัวอย่าง:
     ```hcl
     output "instance_ip" {
       value = aws_instance.example.public_ip
     }
     ```

---

### ตัวอย่างโปรเจคง่าย ๆ
สร้าง EC2 Instance บน AWS:
1. ไฟล์ `main.tf`:
   ```hcl
   provider "aws" {
     region = "us-east-1"
   }

   resource "aws_instance" "my_instance" {
     ami           = "ami-0c55b159cbfafe1f0"
     instance_type = "t2.micro"

     tags = {
       Name = "MyFirstInstance"
     }
   }
   ```
2. รันคำสั่ง:
   ```bash
   terraform init
   terraform plan
   terraform apply
   ```

