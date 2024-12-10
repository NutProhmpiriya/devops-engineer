
# Test Packages for Linux

## **1. `htop`**
- ตัวช่วยแสดงข้อมูลกระบวนการ (process) แบบเรียลไทม์ พร้อม UI ใช้งานง่าย
- **คำสั่งติดตั้ง**:
  ```bash
  sudo apt install htop         # สำหรับ Debian/Ubuntu
  sudo yum install htop         # สำหรับ RHEL/CentOS (ใช้ DNF ในรุ่นใหม่)
  sudo pacman -S htop           # สำหรับ Arch Linux
  ```
- **การใช้งาน**:
  ```bash
  htop
  ```

---

## **2. `tree`**
- ใช้สำหรับแสดงโครงสร้างโฟลเดอร์ในรูปแบบต้นไม้
- **คำสั่งติดตั้ง**:
  ```bash
  sudo apt install tree         # สำหรับ Debian/Ubuntu
  sudo yum install tree         # สำหรับ RHEL/CentOS
  sudo pacman -S tree           # สำหรับ Arch Linux
  ```
- **การใช้งาน**:
  ```bash
  tree
  ```

---

## **3. `cowsay`**
- แพ็คเกจสนุก ๆ ที่แสดงข้อความพร้อมภาพวัวพูด
- **คำสั่งติดตั้ง**:
  ```bash
  sudo apt install cowsay       # สำหรับ Debian/Ubuntu
  sudo yum install cowsay       # สำหรับ RHEL/CentOS
  sudo pacman -S cowsay         # สำหรับ Arch Linux
  ```
- **การใช้งาน**:
  ```bash
  cowsay "Hello, Linux!"
  ```

---

## **4. `fortune`**
- แพ็คเกจที่แสดงคำพูดสุ่มเมื่อรัน
- **คำสั่งติดตั้ง**:
  ```bash
  sudo apt install fortune      # สำหรับ Debian/Ubuntu
  sudo yum install fortune-mod  # สำหรับ RHEL/CentOS
  sudo pacman -S fortune-mod    # สำหรับ Arch Linux
  ```
- **การใช้งาน**:
  ```bash
  fortune
  ```

---

## **5. `neofetch`**
- แสดงข้อมูลระบบในรูปแบบ ASCII art
- **คำสั่งติดตั้ง**:
  ```bash
  sudo apt install neofetch     # สำหรับ Debian/Ubuntu
  sudo yum install neofetch     # สำหรับ RHEL/CentOS
  sudo pacman -S neofetch       # สำหรับ Arch Linux
  ```
- **การใช้งาน**:
  ```bash
  neofetch
  ```

---

## **คำแนะนำสำหรับการทดลอง**
1. **ติดตั้งทีละแพ็คเกจ**: เพื่อตรวจสอบว่าทำงานได้ถูกต้อง
2. **ทดสอบคำสั่งการลบ**: 
   ```bash
   sudo apt remove package_name
   sudo apt autoremove          # ลบแพ็คเกจที่ไม่จำเป็น
   ```
3. **ตรวจสอบแพ็คเกจที่ติดตั้ง**:
   ```bash
   dpkg -l | grep package_name  # สำหรับ Debian/Ubuntu
   rpm -q package_name          # สำหรับ RHEL/CentOS
   pacman -Q package_name       # สำหรับ Arch Linux
   ```
