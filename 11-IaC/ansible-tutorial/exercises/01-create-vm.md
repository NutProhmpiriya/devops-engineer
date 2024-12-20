# Exercise 1: Create and Configure Google Cloud VM Instance
# แบบฝึกหัดที่ 1: สร้างและตั้งค่าเครื่อง VM บน Google Cloud

## Objective | วัตถุประสงค์
Create a new VM instance on Google Cloud using Ansible and install basic web server components.
สร้างเครื่อง VM ใหม่บน Google Cloud โดยใช้ Ansible และติดตั้งคอมโพเนนต์พื้นฐานสำหรับเว็บเซิร์ฟเวอร์

## Requirements | ความต้องการ
1. Google Cloud account with billing enabled
   - บัญชี Google Cloud ที่เปิดใช้งานการเรียกเก็บเงินแล้ว
2. Google Cloud SDK installed
   - ติดตั้ง Google Cloud SDK แล้ว
3. Ansible installed locally
   - ติดตั้ง Ansible บนเครื่องของคุณแล้ว
4. Service account key with appropriate permissions
   - มี Service account key ที่มีสิทธิ์ที่เหมาะสม

## Task | งาน
Create a playbook that:
สร้าง playbook ที่:
1. Creates a new VM instance
   - สร้างเครื่อง VM ใหม่
2. Installs NGINX web server
   - ติดตั้ง NGINX web server
3. Configures basic firewall rules
   - ตั้งค่า firewall rules พื้นฐาน
4. Verifies the web server is running
   - ตรวจสอบว่าเว็บเซิร์ฟเวอร์ทำงานอยู่

## Solution | เฉลย

### 1. Create inventory file (inventory.ini)
### 1. สร้างไฟล์ inventory (inventory.ini)
```ini
[local]
localhost ansible_connection=local
```

### 2. Create variables file (vars.yml)
### 2. สร้างไฟล์ตัวแปร (vars.yml)
```yaml
---
gcp_project: your-project-id
gcp_cred_file: /path/to/service-account.json
machine_type: n1-standard-1
zone: asia-southeast1-a
region: asia-southeast1
instance_name: web-server-1
```

### 3. Create playbook (create_vm.yml)
### 3. สร้าง playbook (create_vm.yml)
```yaml
---
- name: Create GCP VM Instance
  hosts: localhost
  gather_facts: false
  
  vars_files:
    - vars.yml
    
  tasks:
    - name: Create VM instance
      google.cloud.gcp_compute_instance:
        name: "{{ instance_name }}"
        machine_type: "{{ machine_type }}"
        zone: "{{ zone }}"
        project: "{{ gcp_project }}"
        auth_kind: serviceaccount
        service_account_file: "{{ gcp_cred_file }}"
        state: present
        network_interfaces:
          - network: default
            access_configs:
              - name: External NAT
                type: ONE_TO_ONE_NAT
        disks:
          - auto_delete: true
            boot: true
            initialize_params:
              source_image: projects/debian-cloud/global/images/debian-10
        tags:
          items:
            - http-server
            - https-server
      register: gce

    - name: Create firewall rule
      google.cloud.gcp_compute_firewall:
        name: allow-http
        project: "{{ gcp_project }}"
        auth_kind: serviceaccount
        service_account_file: "{{ gcp_cred_file }}"
        network: default
        allowed:
          - ip_protocol: tcp
            ports:
              - '80'
        target_tags:
          - http-server
        state: present

    - name: Wait for SSH to come up
      wait_for:
        host: "{{ gce.networkInterfaces[0].accessConfigs[0].natIP }}"
        port: 22
        delay: 10
        timeout: 300

    - name: Add host to inventory
      add_host:
        name: "{{ gce.networkInterfaces[0].accessConfigs[0].natIP }}"
        groups: webservers
        ansible_user: your_ssh_user

- name: Configure web server
  hosts: webservers
  become: yes
  
  tasks:
    - name: Install NGINX
      apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Start NGINX
      service:
        name: nginx
        state: started
        enabled: yes

    - name: Verify NGINX is running
      uri:
        url: http://localhost
        return_content: yes
      register: webpage
      failed_when: webpage.status != 200
```

### How to Run | วิธีการรัน
1. Save all files in the same directory
   บันทึกไฟล์ทั้งหมดไว้ในไดเรกทอรีเดียวกัน

2. Update vars.yml with your project details
   อัปเดตไฟล์ vars.yml ด้วยรายละเอียดโปรเจคของคุณ

3. Run the playbook:
   รันคำสั่ง:
```bash
ansible-playbook -i inventory.ini create_vm.yml
```

### Expected Output | ผลลัพธ์ที่คาดหวัง
- New VM instance created in GCP
  - สร้างเครื่อง VM ใหม่ใน GCP สำเร็จ
- NGINX installed and running
  - ติดตั้งและรัน NGINX สำเร็จ
- Firewall rules configured
  - ตั้งค่า firewall rules สำเร็จ
- Web server accessible via public IP
  - เข้าถึงเว็บเซิร์ฟเวอร์ผ่าน IP สาธารณะได้

### Verification | การตรวจสอบ
1. Check GCP Console for the new VM instance
   ตรวจสอบเครื่อง VM ใหม่ในคอนโซล GCP

2. Access NGINX welcome page via browser:
   เข้าถึงหน้าต้อนรับ NGINX ผ่านเบราว์เซอร์:
```
http://<VM_PUBLIC_IP>
```

### Cleanup | การทำความสะอาด
To delete the resources, change state: present to state: absent in the playbook and run again.
หากต้องการลบทรัพยากร ให้เปลี่ยน state: present เป็น state: absent ใน playbook แล้วรันอีกครั้ง
