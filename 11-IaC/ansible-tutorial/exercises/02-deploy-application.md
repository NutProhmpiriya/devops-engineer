# Exercise 2: Deploy Application to Google Cloud VM
# แบบฝึกหัดที่ 2: Deploy แอปพลิเคชันไปยังเครื่อง VM บน Google Cloud

## Objective | วัตถุประสงค์
Deploy a simple Python Flask application to a Google Cloud VM instance using Ansible.
Deploy แอปพลิเคชัน Python Flask ไปยังเครื่อง VM บน Google Cloud โดยใช้ Ansible

## Requirements | ความต้องการ
1. Existing GCP VM instance (from Exercise 1)
   - เครื่อง VM บน GCP ที่สร้างจากแบบฝึกหัดที่ 1
2. Python application code
   - โค้ดแอปพลิเคชัน Python
3. Ansible installed locally
   - ติดตั้ง Ansible บนเครื่องของคุณแล้ว

## Task | งาน
Create a playbook that:
สร้าง playbook ที่:
1. Installs Python and pip
   - ติดตั้ง Python และ pip
2. Copies application code to the server
   - คัดลอกโค้ดแอปพลิเคชันไปยังเซิร์ฟเวอร์
3. Installs application dependencies
   - ติดตั้ง dependencies ของแอปพลิเคชัน
4. Configures and starts the application service
   - ตั้งค่าและเริ่มต้นบริการของแอปพลิเคชัน

## Solution | เฉลย

### 1. Create Flask Application (app.py)
### 1. สร้างแอปพลิเคชัน Flask (app.py)
```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return 'Hello from Google Cloud!'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

### 2. Create Requirements File (requirements.txt)
### 2. สร้างไฟล์ Requirements (requirements.txt)
```
Flask==2.0.1
gunicorn==20.1.0
```

### 3. Create Service File (flask_app.service)
### 3. สร้างไฟล์ Service (flask_app.service)
```ini
[Unit]
Description=Flask Application
After=network.target

[Service]
User=www-data
WorkingDirectory=/opt/flask_app
ExecStart=/usr/local/bin/gunicorn --workers 3 --bind unix:flask_app.sock -m 007 app:app

[Install]
WantedBy=multi-user.target
```

### 4. Create Playbook (deploy_app.yml)
### 4. สร้าง Playbook (deploy_app.yml)
```yaml
---
- name: Deploy Flask Application
  hosts: webservers
  become: yes
  
  vars:
    app_dir: /opt/flask_app
    
  tasks:
    - name: Install Python and pip
      apt:
        name: 
          - python3
          - python3-pip
          - python3-venv
        state: present
        update_cache: yes

    - name: Create application directory
      file:
        path: "{{ app_dir }}"
        state: directory
        owner: www-data
        group: www-data
        mode: '0755'

    - name: Copy application files
      copy:
        src: "{{ item.src }}"
        dest: "{{ app_dir }}/{{ item.dest }}"
        owner: www-data
        group: www-data
        mode: '0644'
      with_items:
        - { src: 'app.py', dest: 'app.py' }
        - { src: 'requirements.txt', dest: 'requirements.txt' }

    - name: Install Python packages
      pip:
        requirements: "{{ app_dir }}/requirements.txt"
        virtualenv: "{{ app_dir }}/venv"
        virtualenv_command: python3 -m venv

    - name: Copy service file
      copy:
        src: flask_app.service
        dest: /etc/systemd/system/flask_app.service
        owner: root
        group: root
        mode: '0644'

    - name: Start and enable Flask application service
      systemd:
        name: flask_app
        state: started
        enabled: yes
        daemon_reload: yes

    - name: Configure NGINX
      template:
        src: nginx_config.j2
        dest: /etc/nginx/sites-available/flask_app
      notify: Reload NGINX

    - name: Enable NGINX site
      file:
        src: /etc/nginx/sites-available/flask_app
        dest: /etc/nginx/sites-enabled/flask_app
        state: link
      notify: Reload NGINX

  handlers:
    - name: Reload NGINX
      service:
        name: nginx
        state: reloaded
```

### 5. Create NGINX Configuration Template (nginx_config.j2)
### 5. สร้างเทมเพลตการตั้งค่า NGINX (nginx_config.j2)
```nginx
server {
    listen 80;
    server_name _;

    location / {
        include proxy_params;
        proxy_pass http://unix:/opt/flask_app/flask_app.sock;
    }
}
```

### How to Run | วิธีการรัน
1. Organize your files in the following structure:
   จัดเรียงไฟล์ตามโครงสร้างนี้:
```
deploy/
├── app.py
├── requirements.txt
├── flask_app.service
├── nginx_config.j2
└── deploy_app.yml
```

2. Run the playbook:
   รันคำสั่ง:
```bash
ansible-playbook -i inventory.ini deploy_app.yml
```

### Expected Output | ผลลัพธ์ที่คาดหวัง
- Python and dependencies installed
  - ติดตั้ง Python และ dependencies สำเร็จ
- Application files copied to server
  - คัดลอกไฟล์แอปพลิเคชันไปยังเซิร์ฟเวอร์สำเร็จ
- Flask application running as a service
  - แอปพลิเคชัน Flask ทำงานเป็น service
- NGINX configured as reverse proxy
  - ตั้งค่า NGINX เป็น reverse proxy สำเร็จ

### Verification | การตรวจสอบ
1. Access the application via web browser:
   เข้าถึงแอปพลิเคชันผ่านเบราว์เซอร์:
```
http://<VM_PUBLIC_IP>
```

2. Check application logs:
   ตรวจสอบล็อกของแอปพลิเคชัน:
```bash
sudo journalctl -u flask_app
```

### Troubleshooting | การแก้ไขปัญหา
1. Check service status:
   ตรวจสอบสถานะ service:
```bash
sudo systemctl status flask_app
```

2. Check NGINX status:
   ตรวจสอบสถานะ NGINX:
```bash
sudo systemctl status nginx
```

3. Check NGINX error logs:
   ตรวจสอบล็อกข้อผิดพลาดของ NGINX:
```bash
sudo tail -f /var/log/nginx/error.log
```
