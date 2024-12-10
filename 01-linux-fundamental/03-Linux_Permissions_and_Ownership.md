
# Permissions and Ownership in Linux

## **1. Ownership**
ไฟล์และโฟลเดอร์ใน Linux จะมี **เจ้าของ** 2 กลุ่มหลัก:
1. **User (Owner)**: ผู้ใช้ที่สร้างไฟล์หรือโฟลเดอร์
2. **Group**: กลุ่มของผู้ใช้ที่สามารถเข้าถึงไฟล์หรือโฟลเดอร์ร่วมกันได้
3. **Others**: ผู้ใช้อื่น ๆ ที่ไม่ได้เป็นเจ้าของหรืออยู่ในกลุ่มนั้น

### การตรวจสอบ Ownership
ใช้คำสั่ง `ls -l` เพื่อดูรายละเอียด  
ตัวอย่าง:
```bash
ls -l file.txt
-rw-r--r-- 1 user group 1024 Dec 10 12:00 file.txt
```
- **user**: เจ้าของไฟล์
- **group**: กลุ่มที่ไฟล์นี้สังกัด

### การเปลี่ยน Ownership
1. เปลี่ยนเจ้าของไฟล์:  
   ```bash
   sudo chown new_user file.txt
   ```
2. เปลี่ยนกลุ่มของไฟล์:  
   ```bash
   sudo chgrp new_group file.txt
   ```
3. เปลี่ยนเจ้าของและกลุ่มพร้อมกัน:  
   ```bash
   sudo chown new_user:new_group file.txt
   ```

---

## **2. Permissions**
Permissions กำหนดว่าผู้ใช้สามารถ **อ่าน** (read), **เขียน** (write), หรือ **รัน** (execute) ไฟล์หรือโฟลเดอร์ได้หรือไม่ โดยแบ่งออกเป็น 3 ระดับ:
1. **Owner**: สิทธิ์สำหรับเจ้าของไฟล์
2. **Group**: สิทธิ์สำหรับสมาชิกในกลุ่ม
3. **Others**: สิทธิ์สำหรับผู้ใช้อื่น ๆ

### รูปแบบของ Permissions
Permissions แสดงในรูปแบบ `rwx`:
- **r**: (read) อ่านไฟล์หรือดูเนื้อหาโฟลเดอร์
- **w**: (write) แก้ไขไฟล์หรือเพิ่มไฟล์ในโฟลเดอร์
- **x**: (execute) รันไฟล์หรือเข้าถึงโฟลเดอร์

ตัวอย่าง:
```bash
ls -l file.txt
-rw-r--r-- 1 user group 1024 Dec 10 12:00 file.txt
```
- `-rw-r--r--` แบ่งเป็น 3 ส่วน:
  - **Owner**: `rw-` (อ่านและเขียน)
  - **Group**: `r--` (อ่านอย่างเดียว)
  - **Others**: `r--` (อ่านอย่างเดียว)

---

### การเปลี่ยน Permissions
ใช้คำสั่ง `chmod`:
1. เปลี่ยน Permissions แบบสัญลักษณ์:  
   ```bash
   chmod u+x file.txt   # เพิ่มสิทธิ์ execute ให้ owner
   chmod g-w file.txt   # ลบสิทธิ์ write จาก group
   chmod o=r file.txt   # กำหนด others ให้มีสิทธิ์อ่านเท่านั้น
   ```
2. เปลี่ยน Permissions แบบตัวเลข (Octal):  
   ```bash
   chmod 755 file.txt
   ```
   - **7**: (rwx) = 4 (read) + 2 (write) + 1 (execute)
   - **5**: (r-x) = 4 (read) + 0 (write) + 1 (execute)

---

## **3. Default Permissions (umask)**
- ค่าที่ตั้งไว้สำหรับกำหนดสิทธิ์เริ่มต้นของไฟล์/โฟลเดอร์ใหม่
- ดูค่า `umask` ปัจจุบัน:
  ```bash
  umask
  ```
- ตัวอย่าง:
  - ค่า `umask` = 022:
    - ไฟล์ใหม่: `644` (อ่าน/เขียนสำหรับ owner, อ่านอย่างเดียวสำหรับ group และ others)
    - โฟลเดอร์ใหม่: `755` (เข้าถึงได้โดยทุกคน แต่เขียนได้เฉพาะ owner)

---

## **4. Special Permissions**
1. **Setuid (SUID)**: ให้ไฟล์รันด้วยสิทธิ์ของเจ้าของ  
   ```bash
   chmod u+s file
   ```
   - ตัวอย่าง: `/usr/bin/passwd`

2. **Setgid (SGID)**: ให้ไฟล์หรือโฟลเดอร์รันด้วยสิทธิ์ของกลุ่ม  
   ```bash
   chmod g+s directory
   ```

3. **Sticky Bit**: ป้องกันไม่ให้ผู้ใช้อื่นลบไฟล์ในโฟลเดอร์  
   ```bash
   chmod +t directory
   ```
   - ใช้บ่อยใน `/tmp`

---

## สรุป
การจัดการ **Permissions และ Ownership** เป็นหัวใจสำคัญของความปลอดภัยใน Linux หากตั้งค่าถูกต้องจะช่วยป้องกันการเข้าถึงไฟล์ที่ไม่พึงประสงค์และเพิ่มความยืดหยุ่นในการใช้งานระบบ!
