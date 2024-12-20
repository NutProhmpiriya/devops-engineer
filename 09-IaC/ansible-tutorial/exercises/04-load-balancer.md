# Exercise 4: Configure Load Balancer in Google Cloud
# แบบฝึกหัดที่ 4: ตั้งค่า Load Balancer บน Google Cloud

## Objective | วัตถุประสงค์
Set up a HTTP load balancer in Google Cloud to distribute traffic across multiple VM instances.
ตั้งค่า HTTP load balancer บน Google Cloud เพื่อกระจายการจราจรไปยังเครื่อง VM หลายเครื่อง

## Requirements | ความต้องการ
1. Multiple VM instances running web servers
   - เครื่อง VM หลายเครื่องที่รันเว็บเซิร์ฟเวอร์
2. Google Cloud account with load balancing permissions
   - บัญชี Google Cloud ที่มีสิทธิ์จัดการ load balancing
3. Ansible with Google Cloud modules
   - ติดตั้ง Ansible พร้อมโมดูล Google Cloud

## Task | งาน
Create a playbook that:
สร้าง playbook ที่:
1. Creates an instance group
   - สร้างกลุ่มของเครื่อง VM
2. Configures health checks
   - ตั้งค่าการตรวจสอบสถานะ
3. Sets up load balancing
   - ตั้งค่า load balancing
4. Configures URL maps and forwarding rules
   - ตั้งค่าการ map URL และกฎการส่งต่อ

## Solution | เฉลย

### 1. Create Variables File (vars.yml)
### 1. สร้างไฟล์ตัวแปร (vars.yml)
```yaml
---
gcp_project: your-project-id
gcp_cred_file: /path/to/service-account.json
region: asia-southeast1
zone: asia-southeast1-a
instance_group_name: web-servers
health_check_name: http-basic-check
backend_service_name: web-backend
url_map_name: web-map
target_proxy_name: http-proxy
forwarding_rule_name: http-content-rule
```

### 2. Create Load Balancer Playbook (load_balancer.yml)
### 2. สร้าง Playbook สำหรับ Load Balancer (load_balancer.yml)
```yaml
---
- name: Configure Load Balancer
  hosts: localhost
  gather_facts: false
  
  vars_files:
    - vars.yml
    
  tasks:
    - name: Create instance group
      google.cloud.gcp_compute_instance_group:
        name: "{{ instance_group_name }}"
        zone: "{{ zone }}"
        project: "{{ gcp_project }}"
        auth_kind: serviceaccount
        service_account_file: "{{ gcp_cred_file }}"
        network: default
        instances:
          - "{{ vm1_self_link }}"
          - "{{ vm2_self_link }}"
        named_ports:
          - name: http
            port: 80
        state: present
      register: instance_group

    - name: Create health check
      google.cloud.gcp_compute_health_check:
        name: "{{ health_check_name }}"
        project: "{{ gcp_project }}"
        auth_kind: serviceaccount
        service_account_file: "{{ gcp_cred_file }}"
        type: HTTP
        http_health_check:
          port: 80
          request_path: /
        check_interval_sec: 5
        timeout_sec: 5
        healthy_threshold: 2
        unhealthy_threshold: 2
        state: present
      register: health_check

    - name: Create backend service
      google.cloud.gcp_compute_backend_service:
        name: "{{ backend_service_name }}"
        project: "{{ gcp_project }}"
        auth_kind: serviceaccount
        service_account_file: "{{ gcp_cred_file }}"
        health_checks:
          - "{{ health_check.selfLink }}"
        backends:
          - group: "{{ instance_group.selfLink }}"
            balancing_mode: UTILIZATION
            max_utilization: 0.8
            capacity_scaler: 1.0
        protocol: HTTP
        port_name: http
        enable_cdn: true
        state: present
      register: backend_service

    - name: Create URL map
      google.cloud.gcp_compute_url_map:
        name: "{{ url_map_name }}"
        project: "{{ gcp_project }}"
        auth_kind: serviceaccount
        service_account_file: "{{ gcp_cred_file }}"
        default_service: "{{ backend_service.selfLink }}"
        state: present
      register: url_map

    - name: Create HTTP proxy
      google.cloud.gcp_compute_target_http_proxy:
        name: "{{ target_proxy_name }}"
        project: "{{ gcp_project }}"
        auth_kind: serviceaccount
        service_account_file: "{{ gcp_cred_file }}"
        url_map: "{{ url_map.selfLink }}"
        state: present
      register: http_proxy

    - name: Create forwarding rule
      google.cloud.gcp_compute_global_forwarding_rule:
        name: "{{ forwarding_rule_name }}"
        project: "{{ gcp_project }}"
        auth_kind: serviceaccount
        service_account_file: "{{ gcp_cred_file }}"
        target: "{{ http_proxy.selfLink }}"
        port_range: 80
        ip_protocol: TCP
        load_balancing_scheme: EXTERNAL
        state: present
      register: forwarding_rule

    - name: Display load balancer IP
      debug:
        msg: "Load balancer IP: {{ forwarding_rule.IPAddress }}"
```

### 3. Create Cleanup Playbook (cleanup_lb.yml)
### 3. สร้าง Playbook สำหรับทำความสะอาด (cleanup_lb.yml)
```yaml
---
- name: Cleanup Load Balancer
  hosts: localhost
  gather_facts: false
  
  vars_files:
    - vars.yml
    
  tasks:
    - name: Remove forwarding rule
      google.cloud.gcp_compute_global_forwarding_rule:
        name: "{{ forwarding_rule_name }}"
        project: "{{ gcp_project }}"
        auth_kind: serviceaccount
        service_account_file: "{{ gcp_cred_file }}"
        state: absent

    - name: Remove HTTP proxy
      google.cloud.gcp_compute_target_http_proxy:
        name: "{{ target_proxy_name }}"
        project: "{{ gcp_project }}"
        auth_kind: serviceaccount
        service_account_file: "{{ gcp_cred_file }}"
        state: absent

    - name: Remove URL map
      google.cloud.gcp_compute_url_map:
        name: "{{ url_map_name }}"
        project: "{{ gcp_project }}"
        auth_kind: serviceaccount
        service_account_file: "{{ gcp_cred_file }}"
        state: absent

    - name: Remove backend service
      google.cloud.gcp_compute_backend_service:
        name: "{{ backend_service_name }}"
        project: "{{ gcp_project }}"
        auth_kind: serviceaccount
        service_account_file: "{{ gcp_cred_file }}"
        state: absent

    - name: Remove health check
      google.cloud.gcp_compute_health_check:
        name: "{{ health_check_name }}"
        project: "{{ gcp_project }}"
        auth_kind: serviceaccount
        service_account_file: "{{ gcp_cred_file }}"
        state: absent

    - name: Remove instance group
      google.cloud.gcp_compute_instance_group:
        name: "{{ instance_group_name }}"
        zone: "{{ zone }}"
        project: "{{ gcp_project }}"
        auth_kind: serviceaccount
        service_account_file: "{{ gcp_cred_file }}"
        state: absent
```

### How to Run | วิธีการรัน
1. Update vars.yml with your project details
   อัปเดตไฟล์ vars.yml ด้วยรายละเอียดโปรเจคของคุณ

2. Run the playbook:
   รันคำสั่ง:
```bash
ansible-playbook load_balancer.yml
```

### Expected Output | ผลลัพธ์ที่คาดหวัง
- Instance group created
  - สร้างกลุ่มของเครื่อง VM สำเร็จ
- Health check configured
  - ตั้งค่าการตรวจสอบสถานะสำเร็จ
- Load balancer components created
  - สร้างคอมโพเนนต์ของ load balancer สำเร็จ
- Public IP address assigned
  - กำหนด IP address สาธารณะสำเร็จ

### Verification | การตรวจสอบ
1. Check Google Cloud Console:
   ตรวจสอบในคอนโซล Google Cloud:
   - Network Services > Load Balancing
   - บริการเครือข่าย > Load Balancing

2. Test load balancer:
   ทดสอบ load balancer:
```bash
curl http://<LOAD_BALANCER_IP>
```

3. Monitor traffic distribution:
   ตรวจสอบการกระจายการจราจร:
   - View the monitoring graphs in Cloud Console
   - ดูกราฟการตรวจสอบในคอนโซล Cloud

### Cleanup | การทำความสะอาด
Run the cleanup playbook:
รัน playbook ทำความสะอาด:
```bash
ansible-playbook cleanup_lb.yml
```

### Tips | เคล็ดลับ
1. Always test health checks thoroughly
   ทดสอบการตรวจสอบสถานะอย่างละเอียด
2. Monitor backend service utilization
   ตรวจสอบการใช้งานของ backend service
3. Consider using SSL/TLS for production
   พิจารณาใช้ SSL/TLS สำหรับการใช้งานจริง
4. Implement proper logging and monitoring
   ใส่การบันทึกล็อกและการตรวจสอบที่เหมาะสม
