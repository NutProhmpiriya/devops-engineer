
# Shell Script Basics in Linux

## **1. เริ่มต้นสร้าง Shell Script**
### **1.1 สร้างไฟล์สคริปต์**
สร้างไฟล์ใหม่ด้วยคำสั่ง:
```bash
nano myscript.sh
```

### **1.2 ใส่ Shebang**
บรรทัดแรกของไฟล์ควรเป็น `Shebang` เพื่อระบุว่าใช้ Shell ตัวใดในการรัน:
```bash
#!/bin/bash
```

ตัวอย่าง:
```bash
#!/bin/bash
echo "Hello, World!"
```

### **1.3 ให้สิทธิ์ไฟล์เป็น Executable**
เปลี่ยนสิทธิ์ไฟล์ให้สามารถรันได้:
```bash
chmod +x myscript.sh
```

### **1.4 รันสคริปต์**
รันไฟล์สคริปต์ด้วย:
```bash
./myscript.sh
```

---

## **2. คำสั่งพื้นฐานใน Shell Script**
### **2.1 การแสดงข้อความ**
ใช้คำสั่ง `echo`:
```bash
#!/bin/bash
echo "This is a simple script"
```

### **2.2 การรับค่าจากผู้ใช้**
ใช้คำสั่ง `read`:
```bash
#!/bin/bash
echo "Enter your name:"
read name
echo "Hello, $name!"
```

### **2.3 ตัวแปร (Variables)**
กำหนดและเรียกใช้ตัวแปร:
```bash
#!/bin/bash
name="Lennart"
echo "My name is $name"
```

### **2.4 เงื่อนไข (if-else)**
การใช้เงื่อนไข:
```bash
#!/bin/bash
read -p "Enter a number: " num
if [ $num -gt 10 ]; then
  echo "Number is greater than 10"
else
  echo "Number is less than or equal to 10"
fi
```

### **2.5 การวนซ้ำ (Loops)**
#### **For Loop**:
```bash
#!/bin/bash
for i in 1 2 3 4 5
do
  echo "Number: $i"
done
```

#### **While Loop**:
```bash
#!/bin/bash
count=1
while [ $count -le 5 ]
do
  echo "Count: $count"
  ((count++))
done
```

---

## **3. ฟังก์ชัน (Functions)**
การสร้างฟังก์ชันเพื่อใช้งาน:
```bash
#!/bin/bash
function greet() {
  echo "Hello, $1!"
}

greet "Lennart"
```

---

## **4. ตัวดำเนินการ (Operators)**
### **ตัวอย่างการเปรียบเทียบ**
- `-eq`: เท่ากับ
- `-ne`: ไม่เท่ากับ
- `-gt`: มากกว่า
- `-lt`: น้อยกว่า

ตัวอย่าง:
```bash
#!/bin/bash
a=5
b=10
if [ $a -lt $b ]; then
  echo "$a is less than $b"
fi
```

---

## **5. การจัดการไฟล์**
### ตรวจสอบไฟล์หรือโฟลเดอร์:
```bash
#!/bin/bash
if [ -f myfile.txt ]; then
  echo "File exists"
else
  echo "File does not exist"
fi
```

### ลบไฟล์:
```bash
#!/bin/bash
rm -i myfile.txt
```

---

## **6. การใช้งานพารามิเตอร์**
การส่งค่าผ่านบรรทัดคำสั่ง:
```bash
#!/bin/bash
echo "First argument: $1"
echo "Second argument: $2"
```
รันด้วย:
```bash
./myscript.sh arg1 arg2
```

---

## **7. การ Debug Shell Script**
ใช้ `set -x` เพื่อเปิดโหมด Debug:
```bash
#!/bin/bash
set -x
echo "Debugging this script"
set +x
```

---

## **ตัวอย่าง Shell Script ง่าย ๆ**
### **Backup ไฟล์**
```bash
#!/bin/bash
src="/path/to/source"
dest="/path/to/destination"
backup_name="backup_$(date +%Y%m%d).tar.gz"

tar -czf $dest/$backup_name $src
echo "Backup completed: $backup_name"
```

---

Shell Script มีความยืดหยุ่นและสามารถนำไปประยุกต์ใช้ได้หลากหลาย เช่น การสำรองข้อมูล, การตั้งค่าอัตโนมัติ และการจัดการระบบ
