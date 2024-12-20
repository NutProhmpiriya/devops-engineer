# Introduction to Ansible
# บทนำสู่ Ansible

## What is Ansible?
## Ansible คืออะไร?

Ansible is an open-source automation tool that enables infrastructure as code, configuration management, and application deployment. It's agentless and uses SSH to connect to servers, making it lightweight and easy to get started with.

Ansible เป็นเครื่องมือ automation แบบ open-source ที่ช่วยในการจัดการโครงสร้างพื้นฐานในรูปแบบโค้ด (Infrastructure as Code), การจัดการการตั้งค่า (Configuration Management) และการ Deploy แอปพลิเคชัน โดยไม่ต้องติดตั้ง Agent บนเครื่อง Server และใช้ SSH ในการเชื่อมต่อ ทำให้เบาและเริ่มต้นใช้งานได้ง่าย

## Prerequisites
## สิ่งที่ต้องเตรียม

Before starting with Ansible, ensure you have:
ก่อนเริ่มต้นใช้งาน Ansible คุณต้องมี:

- Python installed (version 2.7 or 3.5+)
  - ติดตั้ง Python (เวอร์ชัน 2.7 หรือ 3.5 ขึ้นไป)
- SSH access to target servers
  - สามารถเข้าถึง server เป้าหมายผ่าน SSH ได้
- Basic understanding of YAML syntax
  - มีความเข้าใจพื้นฐานเกี่ยวกับไวยากรณ์ YAML
- Basic knowledge of Linux commands
  - มีความรู้พื้นฐานเกี่ยวกับคำสั่ง Linux

## Installation
## การติดตั้ง

### On macOS:
### สำหรับ macOS:
```bash
brew install ansible
```

### On Ubuntu/Debian:
### สำหรับ Ubuntu/Debian:
```bash
sudo apt update
sudo apt install ansible
```

### Verify Installation:
### ตรวจสอบการติดตั้ง:
```bash
ansible --version
```

## Basic Concepts
## แนวคิดพื้นฐาน

### 1. Inventory
### 1. Inventory (รายการเครื่อง Server)
- Definition of target hosts and groups
- การกำหนดรายการ host และกลุ่มเป้าหมาย
- Default location: `/etc/ansible/hosts`
- ตำแหน่งไฟล์เริ่มต้น: `/etc/ansible/hosts`
- Can be in INI or YAML format
- สามารถเขียนในรูปแบบ INI หรือ YAML ได้

Example inventory file:
ตัวอย่างไฟล์ inventory:
```ini
[webservers]
web1.example.com
web2.example.com

[databases]
db1.example.com
db2.example.com
```

### 2. Playbooks
### 2. Playbooks (ไฟล์สคริปต์การทำงาน)
- Written in YAML format
- เขียนในรูปแบบ YAML
- Define automation tasks
- กำหนดงานที่ต้องการให้ทำอัตโนมัติ
- Describe the desired state of systems
- อธิบายสถานะที่ต้องการของระบบ

Example playbook:
ตัวอย่าง playbook:
```yaml
---
- name: Update web servers
  hosts: webservers
  become: yes
  
  tasks:
    - name: Ensure Apache is installed
      apt:
        name: apache2
        state: present
    
    - name: Start Apache service
      service:
        name: apache2
        state: started
```

### 3. Modules
### 3. Modules (โมดูลการทำงาน)
- Units of code that Ansible executes
- ส่วนของโค้ดที่ Ansible ใช้ในการทำงาน
- Each module has a specific purpose
- แต่ละโมดูลมีจุดประสงค์เฉพาะ
- Examples: apt, yum, service, copy, file
- ตัวอย่างเช่น: apt, yum, service, copy, file

### 4. Tasks
### 4. Tasks (งานที่ต้องทำ)
- Individual units of work
- หน่วยย่อยของงาน
- Use modules to execute specific actions
- ใช้โมดูลในการทำงานเฉพาะอย่าง
- Can be grouped into plays and playbooks
- สามารถจัดกลุ่มเป็น plays และ playbooks ได้

## First Steps
## ขั้นตอนแรก

1. Create a working directory:
1. สร้างไดเรกทอรีสำหรับทำงาน:
```bash
mkdir ansible-project
cd ansible-project
```

2. Create an inventory file:
2. สร้างไฟล์ inventory:
```bash
touch inventory.ini
```

3. Create your first playbook:
3. สร้าง playbook แรกของคุณ:
```bash
touch playbook.yml
```

## Basic Commands
## คำสั่งพื้นฐาน

1. Ping all hosts:
1. ทดสอบการเชื่อมต่อกับ hosts ทั้งหมด:
```bash
ansible all -m ping -i inventory.ini
```

2. Run a playbook:
2. รันงานใน playbook:
```bash
ansible-playbook playbook.yml -i inventory.ini
```

3. Check host facts:
3. ตรวจสอบข้อมูลของ host:
```bash
ansible hostname -m setup -i inventory.ini
```

## Best Practices
## แนวทางปฏิบัติที่ดี

1. **Directory Structure**
1. **โครงสร้างไดเรกทอรี**
   - Keep playbooks organized in a logical directory structure
   - จัดระเบียบ playbook ในโครงสร้างไดเรกทอรีที่เป็นระบบ
   - Separate variables, tasks, and handlers
   - แยกตัวแปร, tasks และ handlers ออกจากกัน

2. **Version Control**
2. **การควบคุมเวอร์ชัน**
   - Use Git to track changes
   - ใช้ Git ในการติดตามการเปลี่ยนแปลง
   - Include meaningful commit messages
   - เขียนข้อความ commit ที่มีความหมาย

3. **Documentation**
3. **การทำเอกสาร**
   - Comment your code
   - เขียนคำอธิบายในโค้ด
   - Maintain a README file
   - รักษาไฟล์ README ให้เป็นปัจจุบัน
   - Document variables and their purposes
   - ทำเอกสารอธิบายตัวแปรและจุดประสงค์

4. **Security**
4. **ความปลอดภัย**
   - Use Ansible Vault for sensitive data
   - ใช้ Ansible Vault สำหรับข้อมูลที่สำคัญ
   - Regularly update Ansible to the latest version
   - อัปเดต Ansible เป็นเวอร์ชันล่าสุดอย่างสม่ำเสมอ
   - Follow the principle of least privilege
   - ปฏิบัติตามหลักการให้สิทธิ์น้อยที่สุดที่จำเป็น

## Next Steps
## ขั้นตอนต่อไป

After understanding these basics, you can:
หลังจากเข้าใจพื้นฐานเหล่านี้แล้ว คุณสามารถ:
- Learn about roles and how to structure larger projects
- เรียนรู้เกี่ยวกับ roles และวิธีจัดโครงสร้างโปรเจคขนาดใหญ่
- Explore Ansible Galaxy for community-maintained roles
- สำรวจ Ansible Galaxy สำหรับ roles ที่ชุมชนดูแล
- Practice writing more complex playbooks
- ฝึกเขียน playbooks ที่ซับซ้อนมากขึ้น
- Learn about variables and templates
- เรียนรู้เกี่ยวกับตัวแปรและเทมเพลต

## Additional Resources
## แหล่งข้อมูลเพิ่มเติม

- [Official Ansible Documentation](https://docs.ansible.com/)
  - เอกสารอย่างเป็นทางการของ Ansible
- [Ansible Galaxy](https://galaxy.ansible.com/)
  - แหล่งรวม roles ของชุมชน Ansible
- [Ansible GitHub Repository](https://github.com/ansible/ansible)
  - แหล่งโค้ดของ Ansible บน GitHub
