**NGINX** เป็นหนึ่งในซอฟต์แวร์เซิร์ฟเวอร์ที่ได้รับความนิยมสำหรับการจัดการเครือข่ายและบริการโครงสร้างพื้นฐาน มีบทบาทสำคัญในระบบที่ต้องการความยืดหยุ่นและประสิทธิภาพสูง โดยบริการหรือการใช้งานของ NGINX สามารถแบ่งออกได้หลากหลายรูปแบบ เช่น:

---

### **1. Web Server**
- **การใช้งาน:** ใช้เป็นเซิร์ฟเวอร์หลักสำหรับให้บริการเว็บไซต์
- **ตัวอย่าง:**  
  - ใช้สำหรับให้บริการเว็บไซต์ HTML, CSS, และ JavaScript
  - รองรับ HTTPS/SSL
- **การตั้งค่า:**  
  เพิ่มไฟล์ config ไว้ใน `/etc/nginx/sites-available/` และเปิดใช้งานผ่าน symbolic link ไปที่ `/etc/nginx/sites-enabled/`

---

### **2. Reverse Proxy**
- **การใช้งาน:** เป็นพร็อกซีที่รับคำขอจากผู้ใช้แล้วส่งไปยังเซิร์ฟเวอร์ภายใน
- **ตัวอย่าง:**  
  - กระจายคำขอไปยัง Backend Servers เช่น API Server, Application Server
  - ใช้สำหรับโหลดบาลานซ์ (Load Balancing)
- **ข้อดี:** ช่วยเพิ่มความปลอดภัยโดยซ่อนเซิร์ฟเวอร์จริง และปรับปรุงประสิทธิภาพด้วยการ Cache
- **การตั้งค่า:**
```nginx
server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

### **3. Load Balancer**
- **การใช้งาน:** แจกจ่ายคำขอไปยังเซิร์ฟเวอร์หลายตัวเพื่อกระจายภาระ
- **ตัวอย่าง:**  
  - ใช้สำหรับระบบที่มีผู้ใช้จำนวนมาก
  - รองรับการกระจายคำขอแบบ Round Robin, Least Connections หรือ IP Hash
- **การตั้งค่า:**
```nginx
upstream backend {
    server backend1.example.com;
    server backend2.example.com;
}

server {
    listen 80;

    location / {
        proxy_pass http://backend;
    }
}
```

---

### **4. Content Caching**
- **การใช้งาน:** ช่วยลดการโหลดจากเซิร์ฟเวอร์ Backend โดยการเก็บ Cache ไว้ที่ NGINX
- **ตัวอย่าง:**  
  - ใช้สำหรับหน้าเว็บที่ไม่เปลี่ยนแปลงบ่อย
  - ลด Latency และ Bandwidth
- **การตั้งค่า:**
```nginx
location / {
    proxy_cache my_cache;
    proxy_cache_valid 200 10m;
    proxy_pass http://localhost:8080;
}
```

---

### **5. Static Content Hosting**
- **การใช้งาน:** ให้บริการไฟล์ Static เช่น รูปภาพ, CSS, และ JavaScript
- **ตัวอย่าง:**  
  - ใช้ NGINX แทน CDN สำหรับโครงการเล็ก ๆ
- **การตั้งค่า:**
```nginx
server {
    listen 80;
    server_name example.com;

    location /static/ {
        root /var/www/html;
    }
}
```

---

### **6. Security Enhancements**
- **การใช้งาน:** ป้องกันการโจมตี เช่น DDoS หรือ Brute Force
- **ตัวอย่าง:**  
  - ตั้งค่า Rate Limiting เพื่อลดการโจมตีแบบ Request Flood
  - เปิดใช้งาน HTTPS ด้วย Let’s Encrypt
- **การตั้งค่า:**
```nginx
http {
    limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;

    server {
        location / {
            limit_req zone=one burst=5;
        }
    }
}
```

---

### **7. WebSocket Proxying**
- **การใช้งาน:** รองรับการเชื่อมต่อ WebSocket ที่มีการส่งข้อมูลแบบเรียลไทม์
- **ตัวอย่าง:**  
  - ใช้ในแอปพลิเคชัน Chat หรือ Real-time Dashboard
- **การตั้งค่า:**
```nginx
location /ws/ {
    proxy_pass http://localhost:8080;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
}
```

---

### **8. Gzip Compression**
- **การใช้งาน:** บีบอัดข้อมูลเพื่อเพิ่มความเร็วในการโหลดเว็บไซต์
- **การตั้งค่า:**
```nginx
http {
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
}
```

---

### **9. Monitoring and Metrics**
- **การใช้งาน:** เปิดใช้งาน Status Module เพื่อตรวจสอบการทำงานของ NGINX
- **การตั้งค่า:**
```nginx
location /nginx_status {
    stub_status;
    allow 127.0.0.1; # Allow only localhost
    deny all;
}
แน่นอนครับ NGINX มีความสามารถที่หลากหลายมาก และสามารถใช้งานเพิ่มเติมได้ในบริบทต่าง ๆ เช่น:

---

### **10. API Gateway**
- **การใช้งาน:** ใช้ NGINX เป็น API Gateway สำหรับควบคุมและจัดการ API ต่าง ๆ
- **ตัวอย่าง:**  
  - การ Route คำขอตาม Path เช่น `/api/v1` และ `/api/v2`
  - ใช้สำหรับจัดการ Authentication และ Rate Limiting
- **การตั้งค่า:**
```nginx
server {
    listen 80;

    location /api/v1/ {
        proxy_pass http://backend1.example.com;
    }

    location /api/v2/ {
        proxy_pass http://backend2.example.com;
    }
}
```

---

### **11. Dynamic Configuration (NGINX Plus)**
- **การใช้งาน:** ใช้ NGINX Plus ซึ่งเป็นเวอร์ชันพรีเมียมที่รองรับ Dynamic Configuration และ Health Check
- **ตัวอย่าง:**  
  - ใช้สำหรับระบบที่ต้องการเปลี่ยนแปลง Backend Servers โดยไม่ต้อง Restart
- **ฟีเจอร์พิเศษ:**
  - ระบบตรวจสอบสุขภาพของเซิร์ฟเวอร์ (Health Check)
  - ระบบจัดการเซสชัน (Session Persistence)

---

### **12. Integration with Docker**
- **การใช้งาน:** ใช้ NGINX กับ Docker เพื่อจัดการ Proxy และ Load Balancing ระหว่าง Container
- **ตัวอย่าง:**  
  - ใช้ NGINX Proxy Manager หรือ Traefik ร่วมกับ Docker Compose
- **ไฟล์ `docker-compose.yml`:**
```yaml
version: '3'
services:
  nginx:
    image: nginx:latest
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    networks:
      - app-network

  app:
    image: my-app:latest
    networks:
      - app-network

networks:
  app-network:
```

---

### **13. High Availability (HA)**
- **การใช้งาน:** สร้างระบบ NGINX ให้ทำงานร่วมกับ Load Balancer อื่น เช่น Keepalived
- **ตัวอย่าง:**  
  - ใช้ Virtual IP Address (VIP) เพื่อให้ NGINX สลับไปยังเซิร์ฟเวอร์สำรองเมื่อเซิร์ฟเวอร์หลักล้มเหลว

---

### **14. Microservices Architecture**
- **การใช้งาน:** ใช้เป็น Gateway ระหว่าง Microservices
- **ตัวอย่าง:**  
  - ช่วยแยกคำขอที่เข้าไปยัง Service ต่าง ๆ
  - ใช้ร่วมกับ Service Discovery เช่น Consul หรือ etcd
- **การตั้งค่า:**
```nginx
upstream service1 {
    server service1.example.com;
}

upstream service2 {
    server service2.example.com;
}

server {
    listen 80;

    location /service1/ {
        proxy_pass http://service1;
    }

    location /service2/ {
        proxy_pass http://service2;
    }
}
```

---

### **15. Stream (TCP/UDP Proxy)**
- **การใช้งาน:** ใช้สำหรับ Proxy TCP/UDP เช่น Database Connections
- **ตัวอย่าง:**  
  - ใช้สำหรับ MySQL, PostgreSQL, หรือ Redis Proxy
- **การตั้งค่า:**
```nginx
stream {
    upstream database {
        server db1.example.com:3306;
        server db2.example.com:3306;
    }

    server {
        listen 3306;
        proxy_pass database;
    }
}
```

---

### **16. Performance Optimization**
- **การใช้งาน:** ปรับแต่งเพื่อเพิ่มประสิทธิภาพ เช่น ลด Latency และเพิ่ม Throughput
- **การตั้งค่า:**
  - ใช้ Worker Process ตามจำนวน CPU:
    ```nginx
    worker_processes auto;
    ```
  - เพิ่มการตั้งค่า Keep-Alive:
    ```nginx
    http {
        keepalive_timeout 65;
    }
    ```
  - ใช้ Buffer สำหรับคำขอใหญ่:
    ```nginx
    client_body_buffer_size 16k;
    ```

---

### **17. GeoIP Module**
- **การใช้งาน:** กรองคำขอโดยอิงตามตำแหน่งที่ตั้งของผู้ใช้
- **ตัวอย่าง:**  
  - บล็อกการเข้าถึงจากประเทศเฉพาะ
- **การตั้งค่า:**
```nginx
http {
    geo $geoip_country_code {
        default US;
        include /usr/share/GeoIP/geoip.dat;
    }

    server {
        if ($geoip_country_code = CN) {
            return 403;
        }
    }
}
```

---

### **18. WAF (Web Application Firewall)**
- **การใช้งาน:** ใช้ NGINX ร่วมกับ WAF เช่น ModSecurity เพื่อป้องกันการโจมตีเว็บแอปพลิเคชัน
- **ตัวอย่าง:**  
  - ป้องกัน SQL Injection, XSS, และการโจมตีอื่น ๆ

---

### **19. Event Streaming**
- **การใช้งาน:** ใช้สำหรับ Proxy และ Load Balancing ของ Event Streams เช่น RTMP หรือ HTTP/2 Push
- **ตัวอย่าง:**  
  - สร้างระบบ Video Streaming ด้วย NGINX RTMP Module

---

### **20. Custom Error Pages**
- **การใช้งาน:** สร้างหน้าข้อความผิดพลาดเฉพาะ เช่น 404 หรือ 500
- **การตั้งค่า:**
```nginx
server {
    error_page 404 /custom_404.html;
    location = /custom_404.html {
        root /var/www/html;
    }
}
```
