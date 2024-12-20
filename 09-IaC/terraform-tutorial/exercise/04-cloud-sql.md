# Exercise 4: Set up Cloud SQL Instance
# แบบฝึกหัดที่ 4: ติดตั้ง Cloud SQL Instance

## Objective (วัตถุประสงค์)
Create a Cloud SQL instance with proper backup and security configurations.
สร้าง Cloud SQL instance พร้อมการตั้งค่าการสำรองข้อมูลและความปลอดภัยที่เหมาะสม

## Requirements (ข้อกำหนด)
1. Create a MySQL Cloud SQL instance
2. Configure automated backups
3. Set up private IP connectivity
4. Create database and user
5. Configure SSL certificates

## Expected Solution (แนวทางการแก้ปัญหา)

```hcl
# main.tf
provider "google" {
  project = "YOUR_PROJECT_ID"
  region  = "asia-southeast1"
}

# VPC for private IP
resource "google_compute_network" "private_network" {
  name                    = "private-network"
  auto_create_subnetworks = false
}

resource "google_compute_global_address" "private_ip_address" {
  name          = "private-ip-address"
  purpose       = "VPC_PEERING"
  address_type  = "INTERNAL"
  prefix_length = 16
  network       = google_compute_network.private_network.id
}

resource "google_service_networking_connection" "private_vpc_connection" {
  network                 = google_compute_network.private_network.id
  service                 = "servicenetworking.googleapis.com"
  reserved_peering_ranges = [google_compute_global_address.private_ip_address.name]
}

# Cloud SQL instance
resource "google_sql_database_instance" "main" {
  name             = "main-instance"
  database_version = "MYSQL_8_0"
  region           = "asia-southeast1"

  depends_on = [google_service_networking_connection.private_vpc_connection]

  settings {
    tier = "db-f1-micro"
    ip_configuration {
      ipv4_enabled    = false
      private_network = google_compute_network.private_network.id
    }
    
    backup_configuration {
      enabled                        = true
      binary_log_enabled            = true
      start_time                    = "23:00"
      transaction_log_retention_days = 7
      retained_backups              = 7
      retention_unit                = "COUNT"
    }
    
    maintenance_window {
      day          = 7    # Sunday
      hour         = 23
      update_track = "stable"
    }
  }

  deletion_protection = true
}

# Database
resource "google_sql_database" "database" {
  name     = "example-database"
  instance = google_sql_database_instance.main.name
}

# Database user
resource "google_sql_user" "users" {
  name     = "example-user"
  instance = google_sql_database_instance.main.name
  password = var.database_password
}
```

## Variables (ตัวแปร)
```hcl
# variables.tf
variable "database_password" {
  description = "Database user password"
  type        = string
  sensitive   = true
}
```

## Outputs (ผลลัพธ์)
```hcl
# outputs.tf
output "instance_name" {
  value = google_sql_database_instance.main.name
}

output "private_ip_address" {
  value = google_sql_database_instance.main.private_ip_address
}

output "connection_name" {
  value = google_sql_database_instance.main.connection_name
}
```

## Validation (การตรวจสอบ)
1. Verify that the instance is created successfully
2. Check that backups are configured correctly
3. Test private IP connectivity
4. Verify database and user creation
5. Test SSL connection

## Additional Challenge (ความท้าทายเพิ่มเติม)
- Set up read replicas
- Configure point-in-time recovery
- Implement database flags
- Set up monitoring and alerts
- Create automated failover
