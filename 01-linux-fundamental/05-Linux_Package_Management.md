
# Package Management in Linux

## **1. Package Manager ตามดิสโทร**

### **1.1 Debian/Ubuntu และดิสโทรที่เกี่ยวข้อง**
- **APT (Advanced Package Tool)**  
  ใช้สำหรับจัดการแพ็คเกจ `.deb`  
  #### คำสั่งที่สำคัญ:
  ```bash
  sudo apt update            # อัปเดตแหล่งที่มาของแพ็คเกจ
  sudo apt upgrade           # อัปเดตซอฟต์แวร์ทั้งหมด
  sudo apt install package_name   # ติดตั้งแพ็คเกจ
  sudo apt remove package_name    # ลบแพ็คเกจ
  sudo apt autoremove        # ลบแพ็คเกจที่ไม่จำเป็น
  sudo apt search package_name    # ค้นหาแพ็คเกจ
  sudo apt list --installed  # แสดงรายการแพ็คเกจที่ติดตั้ง
  ```

- **dpkg (Debian Package Manager)**  
  ใช้สำหรับจัดการไฟล์ `.deb` โดยตรง  
  #### คำสั่งที่สำคัญ:
  ```bash
  sudo dpkg -i package.deb   # ติดตั้งไฟล์ .deb
  sudo dpkg -r package_name  # ลบแพ็คเกจ
  sudo dpkg -l               # แสดงรายการแพ็คเกจที่ติดตั้ง
  ```

---

### **1.2 RHEL/CentOS และดิสโทรที่เกี่ยวข้อง**
- **YUM (Yellowdog Updater Modified)** (ดิสโทรรุ่นเก่า)  
  #### คำสั่งที่สำคัญ:
  ```bash
  sudo yum update            # อัปเดตซอฟต์แวร์
  sudo yum install package_name   # ติดตั้งแพ็คเกจ
  sudo yum remove package_name    # ลบแพ็คเกจ
  sudo yum list installed     # แสดงรายการแพ็คเกจที่ติดตั้ง
  ```

- **DNF (Dandified Yum)** (ดิสโทรรุ่นใหม่กว่า YUM)  
  #### คำสั่งที่สำคัญ:
  ```bash
  sudo dnf update            # อัปเดตซอฟต์แวร์
  sudo dnf install package_name   # ติดตั้งแพ็คเกจ
  sudo dnf remove package_name    # ลบแพ็คเกจ
  sudo dnf search package_name    # ค้นหาแพ็คเกจ
  sudo dnf list installed     # แสดงรายการแพ็คเกจที่ติดตั้ง
  ```

---

### **1.3 Arch Linux และดิสโทรที่เกี่ยวข้อง**
- **Pacman**  
  ตัวจัดการแพ็คเกจ `.pkg.tar.zst`  
  #### คำสั่งที่สำคัญ:
  ```bash
  sudo pacman -Syu          # อัปเดตซอฟต์แวร์และฐานข้อมูล
  sudo pacman -S package_name    # ติดตั้งแพ็คเกจ
  sudo pacman -R package_name    # ลบแพ็คเกจ
  sudo pacman -Ss package_name   # ค้นหาแพ็คเกจ
  sudo pacman -Qe            # แสดงรายการแพ็คเกจที่ติดตั้งโดยผู้ใช้
  ```

---

### **1.4 openSUSE**
- **Zypper**  
  #### คำสั่งที่สำคัญ:
  ```bash
  sudo zypper refresh       # อัปเดตแหล่งที่มาของแพ็คเกจ
  sudo zypper update        # อัปเดตซอฟต์แวร์
  sudo zypper install package_name   # ติดตั้งแพ็คเกจ
  sudo zypper remove package_name    # ลบแพ็คเกจ
  sudo zypper search package_name    # ค้นหาแพ็คเกจ
  ```

---

## **2. Universal Package Managers**
นอกจาก Package Manager เฉพาะดิสโทร ยังมีตัวจัดการแพ็คเกจที่ใช้ได้กับทุกดิสโทร เช่น:

### **2.1 Snap**
- พัฒนาโดย Canonical ใช้สำหรับติดตั้งซอฟต์แวร์แบบ sandbox  
  #### คำสั่งที่สำคัญ:
  ```bash
  sudo snap install package_name   # ติดตั้งแพ็คเกจ
  sudo snap remove package_name    # ลบแพ็คเกจ
  snap list                       # แสดงรายการแพ็คเกจที่ติดตั้ง
  snap search package_name         # ค้นหาแพ็คเกจ
  ```

### **2.2 Flatpak**
- ใช้สำหรับติดตั้งซอฟต์แวร์แบบ sandbox คล้าย Snap  
  #### คำสั่งที่สำคัญ:
  ```bash
  flatpak install remote_name package_name   # ติดตั้งแพ็คเกจ
  flatpak uninstall package_name            # ลบแพ็คเกจ
  flatpak list                              # แสดงรายการแพ็คเกจที่ติดตั้ง
  flatpak search package_name               # ค้นหาแพ็คเกจ
  ```

### **2.3 AppImage**
- ไฟล์แบบพกพาที่สามารถรันได้โดยไม่ต้องติดตั้ง  
  #### วิธีการใช้งาน:
  ```bash
  chmod +x package.AppImage    # เปลี่ยนไฟล์ให้รันได้
  ./package.AppImage           # รันไฟล์
  ```

---

## **3. คำสั่งที่มีประโยชน์**
- ตรวจสอบขนาดแพ็คเกจ:
  ```bash
  sudo apt show package_name   # สำหรับ APT
  ```
- ดูประวัติการติดตั้งแพ็คเกจ:
  ```bash
  grep install /var/log/dpkg.log   # สำหรับ Debian/Ubuntu
  ```
- ทำความสะอาดระบบ:
  ```bash
  sudo apt autoremove          # ลบแพ็คเกจที่ไม่จำเป็น
  sudo pacman -Sc              # ลบ cache ของ Pacman
  ```

---

## **สรุป**
การจัดการแพ็คเกจใน Linux มีความยืดหยุ่นและแตกต่างกันตามดิสโทร การเลือกใช้ Package Manager ที่เหมาะสมช่วยให้การติดตั้งและอัปเดตซอฟต์แวร์เป็นเรื่องง่ายและปลอดภัย!
