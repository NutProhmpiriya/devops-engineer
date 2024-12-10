**Git** เป็นเครื่องมือสำหรับการควบคุมเวอร์ชันของซอร์สโค้ด (Version Control System) ซึ่งช่วยให้จัดการและติดตามการเปลี่ยนแปลงในโครงการได้ง่ายขึ้น การใช้งาน Git มักใช้ผ่านคำสั่ง (Command) ต่าง ๆ ใน Command Line หรือ Terminal

---

```markdown
# การใช้งาน Git Command

## 1. พื้นฐานของ Git

### **1.1 การตั้งค่าเริ่มต้น**
ก่อนเริ่มใช้งาน Git จำเป็นต้องตั้งค่าข้อมูลผู้ใช้:
```bash
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"
```

- `user.name`: ชื่อของคุณ (แสดงในประวัติการ Commit)
- `user.email`: อีเมลของคุณ

---

## 2. คำสั่งสำคัญของ Git

### **2.1 สร้างหรือ Clone Repository**

#### **สร้าง Repository ใหม่**
```bash
git init
```
- ใช้ในไดเรกทอรีปัจจุบันเพื่อเริ่มต้น Repository

#### **Clone Repository จาก Remote**
```bash
git clone [URL]
```
- ดึงโค้ดจาก Remote Repository เช่น GitHub หรือ GitLab
- ตัวอย่าง:
  ```bash
  git clone https://github.com/username/repository.git
  ```

---

### **2.2 การจัดการไฟล์ใน Repository**

#### **ดูสถานะของ Repository**
```bash
git status
```
- แสดงสถานะไฟล์ใน Repository (ไฟล์ใหม่, ไฟล์ที่เปลี่ยนแปลง)

#### **เพิ่มไฟล์ใน Staging Area**
```bash
git add [filename]
```
- เพิ่มไฟล์ที่ระบุเข้าสู่ Staging Area
- เพิ่มทุกไฟล์ในโฟลเดอร์:
  ```bash
  git add .
  ```

#### **ยกเลิกไฟล์ใน Staging Area**
```bash
git reset [filename]
```
- ย้ายไฟล์ออกจาก Staging Area

---

### **2.3 Commit และ Log**

#### **บันทึกการเปลี่ยนแปลง (Commit)**
```bash
git commit -m "Commit message"
```
- บันทึกการเปลี่ยนแปลงพร้อมข้อความอธิบาย

#### **ดูประวัติ Commit**
```bash
git log
```
- แสดงรายการ Commit ทั้งหมด

#### **ดู Commit แบบย่อ**
```bash
git log --oneline
```

---

### **2.4 การทำงานกับ Branch**

#### **สร้าง Branch ใหม่**
```bash
git branch [branch_name]
```

#### **สลับไปยัง Branch อื่น**
```bash
git checkout [branch_name]
```

#### **สร้าง Branch และสลับไปยัง Branch นั้น**
```bash
git checkout -b [branch_name]
```

#### **ดูรายชื่อ Branch**
```bash
git branch
```

---

### **2.5 การทำงานกับ Remote Repository**

#### **ดู Remote Repository**
```bash
git remote -v
```

#### **เพิ่ม Remote Repository**
```bash
git remote add origin [URL]
```

#### **ดึงการเปลี่ยนแปลงจาก Remote**
```bash
git pull origin [branch_name]
```

#### **ส่งการเปลี่ยนแปลงไปยัง Remote**
```bash
git push origin [branch_name]
```

---

### **2.6 การผสาน Branch (Merge)**

#### **รวม Branch เข้าด้วยกัน**
1. สลับไปยัง Branch ที่ต้องการรวม (เช่น `main`):
   ```bash
   git checkout main
   ```

2. ใช้คำสั่ง Merge:
   ```bash
   git merge [branch_name]
   ```

#### **แก้ไข Conflict**
- เมื่อ Merge แล้วเกิด Conflict ให้แก้ไขไฟล์ที่มีปัญหา จากนั้น:
  ```bash
  git add [filename]
  git commit -m "Resolve merge conflict"
  ```

---

### **2.7 การย้อนกลับ (Undo Changes)**

#### **ยกเลิกการเปลี่ยนแปลงในไฟล์**
```bash
git checkout -- [filename]
```

#### **ยกเลิก Commit ล่าสุด (แต่ไม่ลบการเปลี่ยนแปลง)**
```bash
git reset --soft HEAD~1
```

#### **ยกเลิก Commit และการเปลี่ยนแปลงล่าสุด**
```bash
git reset --hard HEAD~1
```

---

### **2.8 คำสั่งอื่น ๆ ที่สำคัญ**

#### **ดูความแตกต่างของไฟล์**
```bash
git diff
```

#### **ดูรายละเอียดของไฟล์ที่ถูก Commit**
```bash
git show [commit_hash]
```

#### **ลบไฟล์ออกจาก Git**
```bash
git rm [filename]
```

#### **เก็บงานชั่วคราว (Stash)**
```bash
git stash
```
- ใช้เก็บการเปลี่ยนแปลงปัจจุบันเพื่อกลับมาทำงานภายหลัง

---

## 3. ตัวอย่างการใช้งาน Git

### **3.1 สร้างโปรเจกต์และ Push ไปยัง GitHub**
1. สร้าง Repository บน GitHub และคัดลอก URL
2. ใช้คำสั่ง:
   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   git remote add origin [repository_URL]
   git branch -M main
   git push -u origin main
   ```

### **3.2 การทำงานร่วมกัน**
- ใช้ `git pull` เพื่ออัปเดตโค้ดล่าสุด
- ใช้ Branch แยกทำงาน และ Merge เข้าด้วยกันเมื่อเสร็จ

---

## 4. สรุปคำสั่งสำคัญ
| คำสั่ง               | ความหมาย                                |
|----------------------|-----------------------------------------|
| `git init`           | สร้าง Repository ใหม่                  |
| `git clone`          | คัดลอก Repository จาก Remote           |
| `git add`            | เพิ่มไฟล์ไปยัง Staging Area            |
| `git commit`         | บันทึกการเปลี่ยนแปลง                  |
| `git push`           | ส่งการเปลี่ยนแปลงไปยัง Remote         |
| `git pull`           | ดึงการเปลี่ยนแปลงจาก Remote           |
| `git branch`         | จัดการ Branch                          |
| `git checkout`       | สลับไปยัง Branch อื่น                  |
| `git merge`          | รวม Branch                             |
| `git status`         | แสดงสถานะไฟล์                         |
| `git log`            | ดูประวัติ Commit                      |

```

หากต้องการคำอธิบายเพิ่มเติมในส่วนใด เช่นการแก้ Conflict หรือการใช้งาน Git ร่วมกับระบบ CI/CD แจ้งมาได้เลยครับ!