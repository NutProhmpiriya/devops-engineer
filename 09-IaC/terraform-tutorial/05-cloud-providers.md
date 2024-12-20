# Cloud Providers in Terraform
# การใช้งาน Terraform กับผู้ให้บริการ Cloud ต่างๆ

## AWS (Amazon Web Services)

### Provider Configuration
```hcl
provider "aws" {
  region     = "us-west-2"
  access_key = "your-access-key"
  secret_key = "your-secret-key"
}
```

### Common Resources
```hcl
# EC2 Instance
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
}

# S3 Bucket
resource "aws_s3_bucket" "example" {
  bucket = "my-bucket"
}

# VPC
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}
```

## Google Cloud Platform (GCP)

### Provider Configuration
```hcl
provider "google" {
  project     = "my-project-id"
  region      = "us-central1"
  credentials = file("service-account.json")
}
```

### Common Resources
```hcl
# Compute Instance
resource "google_compute_instance" "example" {
  name         = "test-instance"
  machine_type = "e2-medium"
  zone         = "us-central1-a"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
    network = "default"
  }
}

# Cloud Storage Bucket
resource "google_storage_bucket" "example" {
  name     = "my-bucket"
  location = "US"
}
```

## Microsoft Azure

### Provider Configuration
```hcl
provider "azurerm" {
  features {}
  subscription_id = "your-subscription-id"
  client_id       = "your-client-id"
  client_secret   = "your-client-secret"
  tenant_id       = "your-tenant-id"
}
```

### Common Resources
```hcl
# Virtual Machine
resource "azurerm_virtual_machine" "example" {
  name                  = "example-vm"
  location              = "West Europe"
  resource_group_name   = azurerm_resource_group.example.name
  network_interface_ids = [azurerm_network_interface.example.id]
  vm_size              = "Standard_DS1_v2"

  storage_image_reference {
    publisher = "Canonical"
    offer     = "UbuntuServer"
    sku       = "18.04-LTS"
    version   = "latest"
  }
}

# Storage Account
resource "azurerm_storage_account" "example" {
  name                     = "examplesa"
  resource_group_name      = azurerm_resource_group.example.name
  location                 = "West Europe"
  account_tier             = "Standard"
  account_replication_type = "LRS"
}
```

## Multi-Cloud Setup (การตั้งค่าแบบหลาย Cloud)
```hcl
# AWS Provider
provider "aws" {
  region = "us-west-2"
  alias  = "west"
}

# GCP Provider
provider "google" {
  project = "my-project"
  region  = "us-central1"
}

# Azure Provider
provider "azurerm" {
  features {}
}

# Create resources in multiple clouds
resource "aws_instance" "aws_server" {
  provider = aws.west
  # ... configuration ...
}

resource "google_compute_instance" "gcp_server" {
  # ... configuration ...
}

resource "azurerm_virtual_machine" "azure_server" {
  # ... configuration ...
}
```

## Best Practices (แนวทางปฏิบัติที่ดี)

1. การจัดการ Credentials
   - ใช้ environment variables
   - ใช้ service accounts
   - ไม่เก็บ credentials ใน code

2. การแยก State Files
   - แยก state ตาม environment
   - แยก state ตาม cloud provider
   - ใช้ remote state

3. การจัดการ Regions
   - กำหนด default region
   - ใช้ variables สำหรับ region
   - พิจารณาเรื่อง disaster recovery

4. Resource Naming
   - ใช้ prefix ที่ชัดเจน
   - ระบุ environment
   - ใช้ tags อย่างเหมาะสม

5. Cost Management
   - ใช้ cost estimation tools
   - ตั้งค่า budget alerts
   - ทำ cleanup resources ที่ไม่ใช้
