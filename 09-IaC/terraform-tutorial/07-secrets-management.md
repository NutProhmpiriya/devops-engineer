# Secrets Management in Terraform
# การจัดการข้อมูลที่เป็นความลับใน Terraform

## Overview (ภาพรวม)
การจัดการ secrets ใน Terraform เป็นสิ่งสำคัญเพื่อ:
- รักษาความปลอดภัยของข้อมูล
- ป้องกันการรั่วไหลของข้อมูลสำคัญ
- จัดการ credentials อย่างเป็นระบบ

## HashiCorp Vault Integration
### Provider Configuration
```hcl
provider "vault" {
  address = "http://vault.example.com:8200"
  token   = var.vault_token
}

# Read secret from Vault
data "vault_generic_secret" "example" {
  path = "secret/database_credentials"
}

# Use secret in resource
resource "aws_db_instance" "example" {
  username = data.vault_generic_secret.example.data["username"]
  password = data.vault_generic_secret.example.data["password"]
}
```

## AWS Secrets Manager
```hcl
# Read secret from AWS Secrets Manager
data "aws_secretsmanager_secret" "example" {
  name = "database/credentials"
}

data "aws_secretsmanager_secret_version" "example" {
  secret_id = data.aws_secretsmanager_secret.example.id
}

# Use secret in resource
resource "aws_db_instance" "example" {
  username = jsondecode(data.aws_secretsmanager_secret_version.example.secret_string)["username"]
  password = jsondecode(data.aws_secretsmanager_secret_version.example.secret_string)["password"]
}
```

## Azure Key Vault
```hcl
# Read secret from Azure Key Vault
data "azurerm_key_vault_secret" "example" {
  name         = "database-password"
  key_vault_id = azurerm_key_vault.example.id
}

# Use secret in resource
resource "azurerm_sql_server" "example" {
  administrator_login_password = data.azurerm_key_vault_secret.example.value
}
```

## Environment Variables (ตัวแปรสภาพแวดล้อม)
```bash
# Set environment variables
export TF_VAR_db_username="admin"
export TF_VAR_db_password="secret"
```

```hcl
# Use environment variables in Terraform
variable "db_username" {
  type = string
}

variable "db_password" {
  type = string
}

resource "aws_db_instance" "example" {
  username = var.db_username
  password = var.db_password
}
```

## Encrypted Files (ไฟล์ที่เข้ารหัส)
### Using sops
```yaml
# secrets.yaml
db_credentials:
    username: ENC[AES256_GCM,data:......]
    password: ENC[AES256_GCM,data:......]
```

```hcl
# Read encrypted secrets
data "sops_file" "secrets" {
  source_file = "secrets.yaml"
}

# Use decrypted secrets
resource "aws_db_instance" "example" {
  username = data.sops_file.secrets.data["db_credentials.username"]
  password = data.sops_file.secrets.data["db_credentials.password"]
}
```

## Best Practices (แนวทางปฏิบัติที่ดี)

1. การจัดการ Secrets
   - ใช้ secrets management service
   - เข้ารหัสข้อมูลที่สำคัญ
   - แยก secrets ตาม environment

2. Access Control
   - จำกัดการเข้าถึง secrets
   - ใช้ role-based access
   - ทำ audit logging

3. Rotation
   - หมุนเวียน secrets เป็นประจำ
   - ใช้ temporary credentials
   - มีกระบวนการอัพเดท

4. CI/CD Integration
   - ใช้ secure environment variables
   - แยก secrets ออกจาก code
   - ทำ secrets injection

5. Documentation
   - บันทึกวิธีการจัดการ secrets
   - อธิบายกระบวนการ rotation
   - ทำ security guidelines

## Security Guidelines (แนวทางความปลอดภัย)

1. ห้ามเก็บ secrets ใน:
   - Version control
   - Plain text files
   - Public repositories

2. ควรใช้:
   - Encryption at rest
   - Secure transmission
   - Access logging

3. การ Monitor:
   - ติดตามการใช้งาน secrets
   - ตรวจสอบการเข้าถึง
   - แจ้งเตือนเมื่อมีการใช้งานผิดปกติ

## Emergency Procedures (ขั้นตอนฉุกเฉิน)

1. เมื่อ Secrets รั่วไหล:
   - เปลี่ยน secrets ทันที
   - ตรวจสอบการใช้งานผิดปกติ
   - แจ้งผู้เกี่ยวข้อง

2. การกู้คืน:
   - มีแผนสำรอง
   - ทำการ rotate secrets
   - อัพเดทระบบที่เกี่ยวข้อง
