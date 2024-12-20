# Exercise 3: Manage Google Cloud Storage with Ansible
# แบบฝึกหัดที่ 3: จัดการ Google Cloud Storage ด้วย Ansible

## Objective | วัตถุประสงค์
Create and manage Google Cloud Storage buckets and objects using Ansible.
สร้างและจัดการ bucket และออบเจ็กต์ใน Google Cloud Storage โดยใช้ Ansible

## Requirements | ความต้องการ
1. Google Cloud account with Storage permissions
   - บัญชี Google Cloud ที่มีสิทธิ์จัดการ Storage
2. Service account with appropriate roles
   - Service account ที่มีบทบาทที่เหมาะสม
3. Ansible with Google Cloud collection installed
   - ติดตั้ง Ansible พร้อม Google Cloud collection

## Task | งาน
Create a playbook that:
สร้าง playbook ที่:
1. Creates a new storage bucket
   - สร้าง storage bucket ใหม่
2. Uploads files to the bucket
   - อัปโหลดไฟล์ไปยัง bucket
3. Sets bucket permissions
   - ตั้งค่าสิทธิ์ของ bucket
4. Lists bucket contents
   - แสดงรายการเนื้อหาใน bucket

## Solution | เฉลย

### 1. Create Variables File (vars.yml)
### 1. สร้างไฟล์ตัวแปร (vars.yml)
```yaml
---
gcp_project: your-project-id
gcp_cred_file: /path/to/service-account.json
bucket_name: my-ansible-bucket-{{ ansible_date_time.epoch }}
bucket_location: ASIA-SOUTHEAST1
```

### 2. Create Sample Files
### 2. สร้างไฟล์ตัวอย่าง
Create a directory named `files` with some sample content:
สร้างไดเรกทอรีชื่อ `files` พร้อมเนื้อหาตัวอย่าง:
```
files/
├── sample1.txt
├── sample2.txt
└── images/
    ├── image1.jpg
    └── image2.jpg
```

### 3. Create Playbook (manage_storage.yml)
### 3. สร้าง Playbook (manage_storage.yml)
```yaml
---
- name: Manage Google Cloud Storage
  hosts: localhost
  gather_facts: true
  
  vars_files:
    - vars.yml
    
  tasks:
    - name: Create storage bucket
      google.cloud.gcp_storage_bucket:
        name: "{{ bucket_name }}"
        project: "{{ gcp_project }}"
        auth_kind: serviceaccount
        service_account_file: "{{ gcp_cred_file }}"
        location: "{{ bucket_location }}"
        storage_class: STANDARD
        public_access_prevention: enforced
        versioning:
          enabled: true
        state: present
      register: bucket

    - name: Upload files to bucket
      google.cloud.gcp_storage_object:
        bucket: "{{ bucket_name }}"
        name: "{{ item.dest }}"
        src: "{{ item.src }}"
        project: "{{ gcp_project }}"
        auth_kind: serviceaccount
        service_account_file: "{{ gcp_cred_file }}"
        state: present
      with_items:
        - { src: 'files/sample1.txt', dest: 'sample1.txt' }
        - { src: 'files/sample2.txt', dest: 'sample2.txt' }
        - { src: 'files/images/image1.jpg', dest: 'images/image1.jpg' }
        - { src: 'files/images/image2.jpg', dest: 'images/image2.jpg' }

    - name: Set bucket IAM policy
      google.cloud.gcp_storage_bucket_iam_policy:
        bucket: "{{ bucket_name }}"
        project: "{{ gcp_project }}"
        auth_kind: serviceaccount
        service_account_file: "{{ gcp_cred_file }}"
        policy_data:
          bindings:
            - members:
                - "serviceAccount:{{ gcp_service_account }}"
              role: roles/storage.objectViewer
        state: present

    - name: List bucket contents
      google.cloud.gcp_storage_object_info:
        bucket: "{{ bucket_name }}"
        project: "{{ gcp_project }}"
        auth_kind: serviceaccount
        service_account_file: "{{ gcp_cred_file }}"
      register: bucket_contents

    - name: Display bucket contents
      debug:
        msg: "File: {{ item.name }}"
      loop: "{{ bucket_contents.resources }}"
```

### 4. Create Cleanup Playbook (cleanup_storage.yml)
### 4. สร้าง Playbook สำหรับทำความสะอาด (cleanup_storage.yml)
```yaml
---
- name: Cleanup Google Cloud Storage
  hosts: localhost
  gather_facts: false
  
  vars_files:
    - vars.yml
    
  tasks:
    - name: Delete all objects in bucket
      google.cloud.gcp_storage_object:
        bucket: "{{ bucket_name }}"
        name: "{{ item.name }}"
        project: "{{ gcp_project }}"
        auth_kind: serviceaccount
        service_account_file: "{{ gcp_cred_file }}"
        state: absent
      loop: "{{ bucket_contents.resources }}"
      
    - name: Delete bucket
      google.cloud.gcp_storage_bucket:
        name: "{{ bucket_name }}"
        project: "{{ gcp_project }}"
        auth_kind: serviceaccount
        service_account_file: "{{ gcp_cred_file }}"
        state: absent
```

### How to Run | วิธีการรัน
1. Install Google Cloud collection:
   ติดตั้ง Google Cloud collection:
```bash
ansible-galaxy collection install google.cloud
```

2. Create sample files:
   สร้างไฟล์ตัวอย่าง:
```bash
mkdir -p files/images
echo "Sample content 1" > files/sample1.txt
echo "Sample content 2" > files/sample2.txt
# Add some image files to files/images/
```

3. Run the playbook:
   รันคำสั่ง:
```bash
ansible-playbook manage_storage.yml
```

### Expected Output | ผลลัพธ์ที่คาดหวัง
- New storage bucket created
  - สร้าง storage bucket ใหม่สำเร็จ
- Files uploaded to bucket
  - อัปโหลดไฟล์ไปยัง bucket สำเร็จ
- IAM permissions set
  - ตั้งค่าสิทธิ์ IAM สำเร็จ
- List of bucket contents displayed
  - แสดงรายการเนื้อหาใน bucket สำเร็จ

### Verification | การตรวจสอบ
1. Check Google Cloud Console:
   ตรวจสอบในคอนโซล Google Cloud:
   - Navigate to Cloud Storage
   - เข้าไปที่ Cloud Storage
   - Verify bucket exists
   - ตรวจสอบว่ามี bucket อยู่
   - Check uploaded files
   - ตรวจสอบไฟล์ที่อัปโหลด

2. Use gsutil command:
   ใช้คำสั่ง gsutil:
```bash
gsutil ls gs://{{ bucket_name }}
```

### Cleanup | การทำความสะอาด
Run the cleanup playbook to remove all resources:
รัน playbook ทำความสะอาดเพื่อลบทรัพยากรทั้งหมด:
```bash
ansible-playbook cleanup_storage.yml
```

### Tips | เคล็ดลับ
1. Always use unique bucket names
   ใช้ชื่อ bucket ที่ไม่ซ้ำกันเสมอ
2. Be careful with bucket permissions
   ระมัดระวังในการตั้งค่าสิทธิ์ของ bucket
3. Consider using object lifecycle management
   พิจารณาใช้การจัดการวงจรชีวิตของออบเจ็กต์
4. Implement proper error handling
   ใส่การจัดการข้อผิดพลาดที่เหมาะสม
