Ansible เป็นเครื่องมือสำหรับการจัดการระบบ (IT automation) และการปรับใช้งาน (deployment) ที่ใช้สำหรับการจัดการเซิร์ฟเวอร์และโครงสร้างพื้นฐานทางไอทีแบบอัตโนมัติ **โดยไม่ต้องติดตั้ง Agent** บนเซิร์ฟเวอร์ปลายทาง (Agentless) ซึ่งช่วยลดความซับซ้อนและปรับปรุงกระบวนการทำงานในระบบ IT

### คุณสมบัติหลักของ Ansible
1. **ง่ายต่อการใช้งาน**:
   - เขียนการตั้งค่าและกระบวนการทำงานในรูปแบบไฟล์ YAML (เรียกว่า Playbooks) ที่เข้าใจง่าย
2. **Agentless**:
   - ใช้ SSH หรือ WinRM (สำหรับ Windows) ในการสื่อสารกับเซิร์ฟเวอร์ปลายทาง โดยไม่ต้องติดตั้งซอฟต์แวร์เพิ่มเติม
3. **Idempotency**:
   - การดำเนินการจะทำให้ระบบอยู่ในสถานะที่กำหนดไว้เสมอ โดยไม่ทำซ้ำงานที่ไม่จำเป็น
4. **รองรับการจัดการหลากหลายแพลตฟอร์ม**:
   - สามารถใช้จัดการเซิร์ฟเวอร์ที่เป็น Linux, Windows, Cloud Services เช่น AWS, GCP, Azure และอุปกรณ์เครือข่าย

### การใช้งานหลัก
- **Configuration Management**: ตั้งค่าและจัดการเซิร์ฟเวอร์ เช่น การติดตั้งซอฟต์แวร์ การตั้งค่าบริการ
- **Provisioning**: เตรียมระบบหรือเซิร์ฟเวอร์ใหม่ เช่น การสร้าง VM ใน Cloud
- **Application Deployment**: นำแอปพลิเคชันขึ้นระบบ (Deploy)
- **Orchestration**: ประสานงานการทำงานของระบบต่าง ๆ เช่น CI/CD Pipeline

### ตัวอย่างการทำงาน (Playbook)
```yaml
- name: ติดตั้งและเริ่มบริการ Apache
  hosts: webservers
  become: yes
  tasks:
    - name: ติดตั้ง Apache
      apt:
        name: apache2
        state: present

    - name: เริ่มบริการ Apache
      service:
        name: apache2
        state: started
```

การใช้งาน **Ansible** สามารถแบ่งออกเป็นขั้นตอนหลัก ๆ ดังนี้: 

---

### 1. **ติดตั้ง Ansible**
#### บน Linux (Ubuntu/Debian):
```bash
sudo apt update
sudo apt install ansible -y
```

#### บน macOS (ผ่าน Homebrew):
```bash
brew install ansible
```

#### ตรวจสอบเวอร์ชัน:
```bash
ansible --version
```

---

### 2. **เตรียม Inventory**
Inventory คือไฟล์ที่ระบุรายการของเซิร์ฟเวอร์หรือโฮสต์ที่ต้องการจัดการ (โดยปกติจะเป็นไฟล์ `hosts`)

#### ตัวอย่างไฟล์ Inventory (`inventory`):
```ini
[webservers]
192.168.1.10 ansible_user=ubuntu ansible_ssh_private_key_file=/path/to/private_key
192.168.1.11 ansible_user=root

[databases]
192.168.1.20 ansible_user=admin
```

---

### 3. **คำสั่งเบื้องต้น**
- **Ping เพื่อทดสอบการเชื่อมต่อ:**
```bash
ansible all -i inventory -m ping
```

- **ติดตั้งแพ็กเกจ (เช่น nginx):**
```bash
ansible webservers -i inventory -m apt -a "name=nginx state=present" --become
```

- **คัดลอกไฟล์ไปยังเซิร์ฟเวอร์:**
```bash
ansible webservers -i inventory -m copy -a "src=/local/file dest=/remote/path"
```

---

### 4. **เขียน Playbook**
Playbook คือไฟล์ YAML ที่ใช้ระบุชุดคำสั่งหรือกระบวนการที่ต้องการดำเนินการอย่างเป็นขั้นตอน

#### ตัวอย่าง Playbook:
```yaml
- name: ตั้งค่าเซิร์ฟเวอร์ Web
  hosts: webservers
  become: yes
  tasks:
    - name: อัปเดตแพ็กเกจ
      apt:
        update_cache: yes

    - name: ติดตั้ง Nginx
      apt:
        name: nginx
        state: present

    - name: เริ่มบริการ Nginx
      service:
        name: nginx
        state: started
```

#### รัน Playbook:
```bash
ansible-playbook -i inventory playbook.yml
```

---

### 5. **คำสั่งที่ควรรู้**
- **--check**: ตรวจสอบคำสั่งก่อนดำเนินการจริง
```bash
ansible-playbook -i inventory playbook.yml --check
```

- **--tags**: เลือกรันเฉพาะ Tasks ที่ระบุ
```yaml
tasks:
  - name: ติดตั้ง Nginx
    apt:
      name: nginx
      state: present
    tags: nginx
```
```bash
ansible-playbook -i inventory playbook.yml --tags nginx
```

---

### 6. **การจัดการ Role**
Role คือวิธีจัดระเบียบ Playbook เพื่อให้โครงสร้างงานมีความชัดเจนและง่ายต่อการดูแล

#### สร้างโครงสร้าง Role:
```bash
ansible-galaxy init myrole
```

#### โครงสร้าง:
```
myrole/
  tasks/
  handlers/
  templates/
  files/
  vars/
  defaults/
  meta/
```

#### ใช้ Role ใน Playbook:
```yaml
- hosts: webservers
  roles:
    - myrole
```

---

### 7. **การ Debug และตรวจสอบผลลัพธ์**
- **Debug ข้อมูล**:
```yaml
- name: แสดงข้อความ
  debug:
    msg: "Hello, {{ ansible_hostname }}"
```

- **ดู Output ของการรัน**:
```bash
ansible-playbook -i inventory playbook.yml -vvv
```

---

### 8. **แนวทางเพิ่มเติมสำหรับ DevSecOps**
- สร้าง Task เพื่อตรวจสอบไฟร์วอลล์:
```yaml
- name: ตั้งค่าไฟร์วอลล์
  ufw:
    rule: allow
    name: "SSH"
    port: 22
    proto: tcp
```

- ใช้โมดูล Ansible Security เช่น:
  - **ansible-lint** สำหรับตรวจสอบไฟล์ Playbook
  - **Molecule** สำหรับทดสอบ Role

