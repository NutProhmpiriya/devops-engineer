# Exercise 3: Create VPC Network and Subnets
# แบบฝึกหัดที่ 3: สร้าง VPC Network และ Subnets

## Objective (วัตถุประสงค์)
Create a custom VPC network with multiple subnets and firewall rules.
สร้าง VPC network แบบกำหนดเองพร้อมกับ subnets หลายตัวและกฎ firewall

## Requirements (ข้อกำหนด)
1. Create a custom VPC network
2. Create three subnets:
   - Public subnet (10.0.1.0/24)
   - Private subnet (10.0.2.0/24)
   - Database subnet (10.0.3.0/24)
3. Configure firewall rules
4. Set up Cloud NAT for private instances

## Expected Solution (แนวทางการแก้ปัญหา)

```hcl
# main.tf
provider "google" {
  project = "YOUR_PROJECT_ID"
  region  = "asia-southeast1"
}

# VPC Network
resource "google_compute_network" "vpc" {
  name                    = "custom-vpc"
  auto_create_subnetworks = false
}

# Public Subnet
resource "google_compute_subnetwork" "public" {
  name          = "public-subnet"
  ip_cidr_range = "10.0.1.0/24"
  network       = google_compute_network.vpc.id
  region        = "asia-southeast1"
}

# Private Subnet
resource "google_compute_subnetwork" "private" {
  name          = "private-subnet"
  ip_cidr_range = "10.0.2.0/24"
  network       = google_compute_network.vpc.id
  region        = "asia-southeast1"
}

# Database Subnet
resource "google_compute_subnetwork" "database" {
  name          = "database-subnet"
  ip_cidr_range = "10.0.3.0/24"
  network       = google_compute_network.vpc.id
  region        = "asia-southeast1"
}

# Cloud Router
resource "google_compute_router" "router" {
  name    = "vpc-router"
  network = google_compute_network.vpc.id
  region  = "asia-southeast1"
}

# NAT Configuration
resource "google_compute_router_nat" "nat" {
  name                               = "vpc-nat"
  router                            = google_compute_router.router.name
  region                            = google_compute_router.router.region
  nat_ip_allocate_option            = "AUTO_ONLY"
  source_subnetwork_ip_ranges_to_nat = "LIST_OF_SUBNETWORKS"
  
  subnetwork {
    name                    = google_compute_subnetwork.private.id
    source_ip_ranges_to_nat = ["ALL_IP_RANGES"]
  }
}

# Firewall Rules
resource "google_compute_firewall" "allow_internal" {
  name    = "allow-internal"
  network = google_compute_network.vpc.id

  allow {
    protocol = "icmp"
  }
  allow {
    protocol = "tcp"
    ports    = ["0-65535"]
  }
  allow {
    protocol = "udp"
    ports    = ["0-65535"]
  }

  source_ranges = [
    "10.0.1.0/24",
    "10.0.2.0/24",
    "10.0.3.0/24"
  ]
}

resource "google_compute_firewall" "allow_ssh" {
  name    = "allow-ssh"
  network = google_compute_network.vpc.id

  allow {
    protocol = "tcp"
    ports    = ["22"]
  }

  source_ranges = ["0.0.0.0/0"]
  target_tags   = ["ssh"]
}

resource "google_compute_firewall" "allow_db" {
  name    = "allow-db"
  network = google_compute_network.vpc.id

  allow {
    protocol = "tcp"
    ports    = ["3306"]
  }

  source_ranges = ["10.0.2.0/24"]  # Only from private subnet
  target_tags   = ["database"]
}
```

## Outputs (ผลลัพธ์)
```hcl
# outputs.tf
output "vpc_id" {
  value = google_compute_network.vpc.id
}

output "public_subnet_id" {
  value = google_compute_subnetwork.public.id
}

output "private_subnet_id" {
  value = google_compute_subnetwork.private.id
}

output "database_subnet_id" {
  value = google_compute_subnetwork.database.id
}
```

## Validation (การตรวจสอบ)
1. Verify that all subnets are created correctly
2. Test connectivity between subnets
3. Verify that NAT is working for private instances
4. Test firewall rules

## Additional Challenge (ความท้าทายเพิ่มเติม)
- Add VPC peering with another VPC
- Implement custom routes
- Set up VPN gateway
- Create subnet flow logs
