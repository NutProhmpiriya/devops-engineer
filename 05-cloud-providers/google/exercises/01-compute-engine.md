# แบบฝึกหัดที่ 1: การตั้งค่าและจัดการ Compute Engine
# Exercise 1: Setting up and Managing Compute Engine

## โจทย์ | Problem
สร้างและกำหนดค่า Virtual Machine บน Google Compute Engine พร้อมติดตั้ง Web Server สำหรับรันแอปพลิเคชัน Node.js โดยมีความต้องการดังนี้:

1. สร้าง VM with the following specifications:
   - Machine type: e2-medium
   - Region: asia-southeast1 (Singapore)
   - OS: Ubuntu 20.04 LTS
   - Boot disk size: 20GB
   - Allow HTTP/HTTPS traffic

2. ติดตั้ง Node.js version 16.x และ Nginx
3. สร้าง firewall rule เพื่อเปิด port 3000
4. สร้าง startup script เพื่อรัน Node.js application อัตโนมัติเมื่อ VM เริ่มทำงาน

## เฉลย | Solution

### 1. สร้าง VM ด้วย gcloud command:
```bash
# Create the VM
gcloud compute instances create nodejs-server \
    --machine-type=e2-medium \
    --zone=asia-southeast1-a \
    --image-family=ubuntu-2004-lts \
    --image-project=ubuntu-os-cloud \
    --boot-disk-size=20GB \
    --tags=http-server,https-server,nodejs-app \
    --metadata-from-file=startup-script=startup.sh
```

### 2. ติดตั้ง Node.js และ Nginx:
```bash
# Connect to VM
gcloud compute ssh nodejs-server --zone=asia-southeast1-a

# Install Node.js
curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
sudo apt-get install -y nodejs

# Install Nginx
sudo apt-get update
sudo apt-get install -y nginx
```

### 3. สร้าง Firewall Rule:
```bash
# Create firewall rule for Node.js application
gcloud compute firewall-rules create allow-nodejs \
    --allow tcp:3000 \
    --target-tags=nodejs-app \
    --description="Allow Node.js application traffic"
```

### 4. สร้าง Startup Script (startup.sh):
```bash
#!/bin/bash
# Install Node.js
curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
sudo apt-get install -y nodejs

# Install Nginx
sudo apt-get update
sudo apt-get install -y nginx

# Create sample Node.js application
mkdir -p /opt/nodeapp
cat > /opt/nodeapp/app.js << 'EOF'
const http = require('http');
const server = http.createServer((req, res) => {
    res.writeHead(200);
    res.end('Hello from Google Cloud!');
});
server.listen(3000);
EOF

# Create systemd service
cat > /etc/systemd/system/nodeapp.service << 'EOF'
[Unit]
Description=Node.js Application
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/opt/nodeapp
ExecStart=/usr/bin/node app.js
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF

# Start the application
systemctl enable nodeapp
systemctl start nodeapp
```

## การทดสอบ | Testing
1. ตรวจสอบสถานะ VM:
```bash
gcloud compute instances describe nodejs-server --zone=asia-southeast1-a
```

2. ทดสอบการเข้าถึงแอปพลิเคชัน:
```bash
# Get the external IP
export VM_IP=$(gcloud compute instances describe nodejs-server --zone=asia-southeast1-a --format='get(networkInterfaces[0].accessConfigs[0].natIP)')

# Test the application
curl http://$VM_IP:3000
```

## เพิ่มเติม | Additional Notes
- ควรใช้ Instance Templates สำหรับการสร้าง VM ในสภาพแวดล้อมจริง
- พิจารณาใช้ Managed Instance Groups สำหรับ high availability
- ใช้ Cloud Storage หรือ Cloud Source Repositories สำหรับเก็บ application code
- ตั้งค่า monitoring และ logging ด้วย Cloud Operations Suite
