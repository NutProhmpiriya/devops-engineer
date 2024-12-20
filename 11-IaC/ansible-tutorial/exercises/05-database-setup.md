# Exercise 5: Setup Cloud SQL Database with Ansible
# แบบฝึกหัดที่ 5: ตั้งค่าฐานข้อมูล Cloud SQL ด้วย Ansible

## Objective | วัตถุประสงค์
Create and configure a Cloud SQL instance using Ansible, including database setup and user management.
สร้างและตั้งค่าอินสแตนซ์ Cloud SQL โดยใช้ Ansible รวมถึงการตั้งค่าฐานข้อมูลและการจัดการผู้ใช้

## Requirements | ความต้องการ
1. Google Cloud account with Cloud SQL permissions
   - บัญชี Google Cloud ที่มีสิทธิ์จัดการ Cloud SQL
2. Service account with appropriate roles
   - Service account ที่มีบทบาทที่เหมาะสม
3. Ansible with Google Cloud collection
   - ติดตั้ง Ansible พร้อม Google Cloud collection

## Task | งาน
Create a playbook that:
สร้าง playbook ที่:
1. Creates a Cloud SQL instance
   - สร้างอินสแตนซ์ Cloud SQL
2. Configures database settings
   - ตั้งค่าฐานข้อมูล
3. Creates databases and users
   - สร้างฐานข้อมูลและผู้ใช้
4. Sets up backup configuration
   - ตั้งค่าการสำรองข้อมูล

## Solution | เฉลย

### 1. Create Variables File (vars.yml)
### 1. สร้างไฟล์ตัวแปร (vars.yml)
```yaml
---
gcp_project: your-project-id
gcp_cred_file: /path/to/service-account.json
region: asia-southeast1
instance_name: my-sql-instance
database_version: MYSQL_8_0
tier: db-f1-micro
storage_size_gb: 10
database_name: application_db
database_user: app_user
database_password: "{{ lookup('password', '/dev/null length=16 chars=ascii_letters,digits') }}"
```

### 2. Create Database Setup Playbook (setup_database.yml)
### 2. สร้าง Playbook สำหรับตั้งค่าฐานข้อมูล (setup_database.yml)
```yaml
---
- name: Setup Cloud SQL Database
  hosts: localhost
  gather_facts: false
  
  vars_files:
    - vars.yml
    
  tasks:
    - name: Create Cloud SQL instance
      google.cloud.gcp_sql_instance:
        name: "{{ instance_name }}"
        project: "{{ gcp_project }}"
        auth_kind: serviceaccount
        service_account_file: "{{ gcp_cred_file }}"
        database_version: "{{ database_version }}"
        region: "{{ region }}"
        settings:
          tier: "{{ tier }}"
          ip_configuration:
            authorized_networks:
              - name: allow-all
                value: "0.0.0.0/0"
          backup_configuration:
            enabled: true
            start_time: "23:00"
          storage_auto_resize: true
          storage_size_gb: "{{ storage_size_gb }}"
        state: present
      register: sql_instance

    - name: Create database
      google.cloud.gcp_sql_database:
        name: "{{ database_name }}"
        project: "{{ gcp_project }}"
        auth_kind: serviceaccount
        service_account_file: "{{ gcp_cred_file }}"
        instance: "{{ instance_name }}"
        charset: utf8mb4
        collation: utf8mb4_unicode_ci
        state: present
      register: database

    - name: Create database user
      google.cloud.gcp_sql_user:
        name: "{{ database_user }}"
        project: "{{ gcp_project }}"
        auth_kind: serviceaccount
        service_account_file: "{{ gcp_cred_file }}"
        instance: "{{ instance_name }}"
        host: "%"
        password: "{{ database_password }}"
        state: present
      register: db_user

    - name: Save database connection information
      copy:
        content: |
          DATABASE_HOST={{ sql_instance.ipAddresses[0].ipAddress }}
          DATABASE_NAME={{ database_name }}
          DATABASE_USER={{ database_user }}
          DATABASE_PASSWORD={{ database_password }}
        dest: ./database_credentials.env
        mode: '0600'

    - name: Display connection information
      debug:
        msg:
          - "Database Instance: {{ instance_name }}"
          - "Host: {{ sql_instance.ipAddresses[0].ipAddress }}"
          - "Database: {{ database_name }}"
          - "User: {{ database_user }}"
          - "Password saved in database_credentials.env"
```

### 3. Create Database Schema (schema.sql)
### 3. สร้างสคีมาฐานข้อมูล (schema.sql)
```sql
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE products (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    price DECIMAL(10,2) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 4. Create Schema Application Playbook (apply_schema.yml)
### 4. สร้าง Playbook สำหรับใช้งานสคีมา (apply_schema.yml)
```yaml
---
- name: Apply Database Schema
  hosts: localhost
  gather_facts: false
  
  vars_files:
    - vars.yml
    
  tasks:
    - name: Install MySQL client
      apt:
        name: mysql-client
        state: present
      become: yes

    - name: Apply database schema
      mysql_db:
        login_host: "{{ sql_instance.ipAddresses[0].ipAddress }}"
        login_user: "{{ database_user }}"
        login_password: "{{ database_password }}"
        name: "{{ database_name }}"
        state: import
        target: schema.sql
```

### 5. Create Cleanup Playbook (cleanup_database.yml)
### 5. สร้าง Playbook สำหรับทำความสะอาด (cleanup_database.yml)
```yaml
---
- name: Cleanup Cloud SQL Database
  hosts: localhost
  gather_facts: false
  
  vars_files:
    - vars.yml
    
  tasks:
    - name: Delete database user
      google.cloud.gcp_sql_user:
        name: "{{ database_user }}"
        project: "{{ gcp_project }}"
        auth_kind: serviceaccount
        service_account_file: "{{ gcp_cred_file }}"
        instance: "{{ instance_name }}"
        state: absent

    - name: Delete database
      google.cloud.gcp_sql_database:
        name: "{{ database_name }}"
        project: "{{ gcp_project }}"
        auth_kind: serviceaccount
        service_account_file: "{{ gcp_cred_file }}"
        instance: "{{ instance_name }}"
        state: absent

    - name: Delete Cloud SQL instance
      google.cloud.gcp_sql_instance:
        name: "{{ instance_name }}"
        project: "{{ gcp_project }}"
        auth_kind: serviceaccount
        service_account_file: "{{ gcp_cred_file }}"
        state: absent
```

### How to Run | วิธีการรัน
1. Update vars.yml with your project details
   อัปเดตไฟล์ vars.yml ด้วยรายละเอียดโปรเจคของคุณ

2. Create the database instance:
   สร้างอินสแตนซ์ฐานข้อมูล:
```bash
ansible-playbook setup_database.yml
```

3. Apply the schema:
   ใช้งานสคีมา:
```bash
ansible-playbook apply_schema.yml
```

### Expected Output | ผลลัพธ์ที่คาดหวัง
- Cloud SQL instance created
  - สร้างอินสแตนซ์ Cloud SQL สำเร็จ
- Database and user created
  - สร้างฐานข้อมูลและผู้ใช้สำเร็จ
- Schema applied
  - ใช้งานสคีมาสำเร็จ
- Connection information saved
  - บันทึกข้อมูลการเชื่อมต่อสำเร็จ

### Verification | การตรวจสอบ
1. Check Google Cloud Console:
   ตรวจสอบในคอนโซล Google Cloud:
   - SQL > Instance details
   - SQL > รายละเอียดอินสแตนซ์

2. Connect to database:
   เชื่อมต่อกับฐานข้อมูล:
```bash
mysql -h <DATABASE_HOST> -u <DATABASE_USER> -p
```

3. Verify tables:
   ตรวจสอบตาราง:
```sql
USE application_db;
SHOW TABLES;
```

### Cleanup | การทำความสะอาด
Run the cleanup playbook:
รัน playbook ทำความสะอาด:
```bash
ansible-playbook cleanup_database.yml
```

### Tips | เคล็ดลับ
1. Always backup before schema changes
   สำรองข้อมูลก่อนเปลี่ยนแปลงสคีมาเสมอ
2. Use strong passwords
   ใช้รหัสผ่านที่รัดกุม
3. Limit network access appropriately
   จำกัดการเข้าถึงเครือข่ายอย่างเหมาะสม
4. Monitor database performance
   ตรวจสอบประสิทธิภาพของฐานข้อมูล
5. Implement proper backup strategy
   ใช้กลยุทธ์การสำรองข้อมูลที่เหมาะสม
