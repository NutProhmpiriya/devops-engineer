# Exercise 1: Create a Basic Compute Instance
# แบบฝึกหัดที่ 1: สร้าง Compute Instance พื้นฐาน

## Objective (วัตถุประสงค์)
Create a basic Google Compute Engine instance using Terraform.
สร้าง Google Compute Engine instance พื้นฐานโดยใช้ Terraform

## Requirements (ข้อกำหนด)
1. Create a VM instance with the following specifications:
   - Name: "exercise-instance"
   - Machine type: "e2-medium"
   - Zone: "asia-southeast1-a"
   - Boot disk image: "debian-cloud/debian-11"
   - Network tags: ["http-server", "https-server"]

2. Allow HTTP and HTTPS traffic through firewall rules

## Expected Solution (แนวทางการแก้ปัญหา)

```hcl
# main.tf
provider "google" {
  project = "YOUR_PROJECT_ID"
  region  = "asia-southeast1"
}

resource "google_compute_instance" "exercise_instance" {
  name         = "exercise-instance"
  machine_type = "e2-medium"
  zone         = "asia-southeast1-a"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }

  network_interface {
    network = "default"
    access_config {
      // Ephemeral public IP
    }
  }

  tags = ["http-server", "https-server"]
}

resource "google_compute_firewall" "http" {
  name    = "allow-http"
  network = "default"

  allow {
    protocol = "tcp"
    ports    = ["80", "443"]
  }

  source_ranges = ["0.0.0.0/0"]
  target_tags   = ["http-server", "https-server"]
}
```

## Validation (การตรวจสอบ)
After applying the configuration:
1. Verify that the instance is running in the GCP Console
2. Verify that HTTP/HTTPS traffic is allowed
3. Try to connect to the instance via SSH

## Additional Challenge (ความท้าทายเพิ่มเติม)
- Add a startup script that installs nginx
- Create a custom network instead of using the default network
- Add labels to the instance for better organization
