แน่นอน! นี่คือข้อมูลเพิ่มเติมเกี่ยวกับ Docker:

---

### ฟีเจอร์เพิ่มเติมของ Docker
1. **Docker Compose**:
   - ใช้จัดการหลายคอนเทนเนอร์พร้อมกัน
   - กำหนดการตั้งค่าด้วยไฟล์ `docker-compose.yml`
   - ตัวอย่างไฟล์ `docker-compose.yml` สำหรับแอปพลิเคชันที่มี Web Server และ Database:
     ```yaml
     version: '3.8'
     services:
       web:
         image: nginx
         ports:
           - "8080:80"
       db:
         image: mysql
         environment:
           MYSQL_ROOT_PASSWORD: example
     ```
     จากนั้นใช้คำสั่ง:
     ```bash
     docker-compose up
     ```

2. **Docker Volume**:
   - ใช้จัดการข้อมูลถาวรของคอนเทนเนอร์
   - ตัวอย่าง:
     ```bash
     docker run -v /path/on/host:/path/in/container myapp
     ```
   - Volume จะช่วยให้ข้อมูลใน container ไม่หายเมื่อ container ถูกลบ

3. **Docker Network**:
   - ใช้สำหรับการสื่อสารระหว่างคอนเทนเนอร์
   - ตัวอย่าง:
     ```bash
     docker network create my_network
     docker run --network my_network --name web nginx
     docker run --network my_network --name db mysql
     ```
   - คอนเทนเนอร์ `web` และ `db` จะสามารถติดต่อกันได้ผ่านชื่อของคอนเทนเนอร์

---

### คำสั่ง Docker ที่มีประโยชน์
1. **ดูข้อมูลระบบ Docker**:
   ```bash
   docker info
   ```

2. **ดู Image ทั้งหมดในเครื่อง**:
   ```bash
   docker images
   ```

3. **ลบ Image**:
   ```bash
   docker rmi <image_id>
   ```

4. **ลบคอนเทนเนอร์ที่หยุดทำงานแล้ว**:
   ```bash
   docker container prune
   ```

5. **ลบคอนเทนเนอร์ทั้งหมด (ทั้งที่รันและหยุด)**:
   ```bash
   docker rm $(docker ps -aq)
   ```

6. **ดู Log ของคอนเทนเนอร์**:
   ```bash
   docker logs <container_id>
   ```

---

### Docker Registry และ Docker Hub
1. **Docker Hub**:
   - เป็นแพลตฟอร์มที่เก็บและแชร์ Docker Images
   - ค้นหา Image สำเร็จรูป เช่น MySQL, Nginx, Node.js ได้จาก Docker Hub
   - ตัวอย่าง:
     ```bash
     docker pull mysql
     ```

2. **สร้าง Registry ส่วนตัว**:
   - สามารถสร้าง Registry ภายในองค์กรได้
   - ตัวอย่างการตั้งค่า Registry ส่วนตัว:
     ```bash
     docker run -d -p 5000:5000 --name registry registry:2
     ```
   - ใช้คำสั่ง:
     ```bash
     docker tag myapp localhost:5000/myapp
     docker push localhost:5000/myapp
     ```

---

### แนวทางการใช้ Docker ในการพัฒนา
1. **Environment Consistency**:
   - ใช้ Docker เพื่อให้ทีมพัฒนามีสภาพแวดล้อมเดียวกัน
   - ลดปัญหา "มันรันได้ในเครื่องฉัน แต่ไม่ได้ในเครื่องคุณ"

2. **CI/CD Integration**:
   - ใช้ Docker ในขั้นตอน Continuous Integration และ Continuous Deployment
   - เช่น รันเทสต์ใน container หรือ deploy image โดยตรงไปยัง production

3. **Microservices**:
   - Docker เหมาะสำหรับการพัฒนา Microservices เพราะแต่ละบริการสามารถรันใน container แยกกัน

---

### แนวทางปฏิบัติที่ดี (Best Practices)
1. ใช้ **Multi-stage Builds** ใน Dockerfile เพื่อลดขนาดของ Image:
   ```Dockerfile
   # Stage 1: Build
   FROM node:16 AS builder
   WORKDIR /app
   COPY . .
   RUN npm install && npm run build

   # Stage 2: Production
   FROM nginx:alpine
   COPY --from=builder /app/build /usr/share/nginx/html
   ```

2. ใช้ **.dockerignore** เพื่อไม่ให้ไฟล์ที่ไม่จำเป็นถูกคัดลอกเข้าไปใน Image:
   ```plaintext
   node_modules
   *.log
   .env
   ```

3. ใช้ Tag แบบระบุเวอร์ชันเสมอ:
   ```bash
   docker build -t myapp:1.0 .
   ```

4. หมั่นลบ Image และคอนเทนเนอร์ที่ไม่ได้ใช้งาน:
   ```bash
   docker system prune
   ```

😊