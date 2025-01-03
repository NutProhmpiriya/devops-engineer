
# File System Hierarchy in Linux

## **1. `/` (Root)**
- โฟลเดอร์หลักของระบบไฟล์
- โฟลเดอร์ย่อยทั้งหมดจะอยู่ภายใต้ `/`
- ใช้เป็นจุดเริ่มต้นสำหรับระบบไฟล์ทั้งหมด

---

## **2. `/bin`**
- เก็บคำสั่งพื้นฐานที่ผู้ใช้ทั่วไปสามารถรันได้ เช่น `ls`, `cp`, `mv`, `cat`
- ใช้งานสำหรับคำสั่งที่จำเป็นในระบบ

---

## **3. `/sbin`**
- เก็บคำสั่งระบบที่ผู้ดูแลระบบ (root) ใช้งาน เช่น `reboot`, `ifconfig`, `fsck`
- ผู้ใช้ทั่วไปไม่สามารถเข้าถึงได้โดยตรง

---

## **4. `/boot`**
- เก็บไฟล์ที่เกี่ยวข้องกับการบูตระบบ เช่น kernel, bootloader (เช่น GRUB)
- **สำคัญมาก** ห้ามแก้ไขโดยไม่รู้จักโครงสร้างของไฟล์

---

## **5. `/dev`**
- เก็บไฟล์ที่แทนฮาร์ดแวร์และอุปกรณ์ เช่น ฮาร์ดดิสก์ (`/dev/sda`), USB (`/dev/ttyUSB0`)
- ใช้สำหรับการติดต่อกับอุปกรณ์โดยตรง

---

## **6. `/etc`**
- เก็บไฟล์ตั้งค่าของระบบและแอปพลิเคชัน เช่น การตั้งค่าเครือข่าย (`/etc/network`), ไฟล์ตั้งค่า `passwd`, `hosts`
- ไฟล์ในนี้จะถูกแก้ไขเมื่อมีการตั้งค่าระบบ

---

## **7. `/home`**
- โฟลเดอร์ของผู้ใช้แต่ละคน เช่น `/home/user1`, `/home/user2`
- เก็บไฟล์ส่วนตัวและการตั้งค่าของผู้ใช้

---

## **8. `/lib`**
- เก็บไฟล์ไลบรารีที่ใช้โดยโปรแกรมใน `/bin` และ `/sbin`
- ไลบรารีเหล่านี้จำเป็นสำหรับการทำงานของระบบ

---

## **9. `/media`**
- จุดเชื่อมต่อ (mount points) สำหรับอุปกรณ์ภายนอก เช่น USB, CD/DVD
- ตัวอย่างเช่น `/media/usb_drive`

---

## **10. `/mnt`**
- จุดเชื่อมต่อชั่วคราวที่ใช้ mount อุปกรณ์ภายนอก
- ส่วนใหญ่จะใช้โดยผู้ดูแลระบบสำหรับการ mount แบบชั่วคราว

---

## **11. `/opt`**
- เก็บซอฟต์แวร์เพิ่มเติมที่ไม่ได้มาพร้อมกับระบบ เช่น แอปพลิเคชันที่ติดตั้งจากภายนอก
- ใช้สำหรับแพ็คเกจที่เป็น third-party

---

## **12. `/proc`**
- เก็บข้อมูลระบบในรูปแบบเสมือน (virtual filesystem)
- ใช้ดูข้อมูลเกี่ยวกับกระบวนการ (process) และฮาร์ดแวร์ เช่น `/proc/cpuinfo`, `/proc/meminfo`

---

## **13. `/root`**
- โฟลเดอร์บ้านของผู้ใช้ `root`
- ใช้เก็บไฟล์ส่วนตัวของผู้ดูแลระบบ

---

## **14. `/run`**
- เก็บข้อมูลระบบชั่วคราวที่ต้องใช้ในช่วงที่ระบบบูต
- ข้อมูลจะถูกลบเมื่อระบบรีบูต

---

## **15. `/srv`**
- เก็บข้อมูลบริการต่าง ๆ ของระบบ เช่น เว็บเซิร์ฟเวอร์ (`/srv/www`)

---

## **16. `/tmp`**
- เก็บไฟล์ชั่วคราวที่สร้างโดยระบบหรือโปรแกรมต่าง ๆ
- ไฟล์ในนี้มักจะถูกลบเมื่อรีบูต

---

## **17. `/usr`**
- เก็บโปรแกรมและไฟล์ที่ใช้โดยผู้ใช้ทั่วไป
  - `/usr/bin` - โปรแกรมทั่วไป เช่น `vim`, `git`
  - `/usr/lib` - ไลบรารีที่ใช้โดยโปรแกรมใน `/usr/bin`
  - `/usr/share` - ไฟล์ที่แชร์ เช่น ไอคอน ธีม เอกสารช่วยเหลือ

---

## **18. `/var`**
- เก็บข้อมูลที่เปลี่ยนแปลงบ่อย เช่น logs (`/var/log`), spool files (`/var/spool`), cache (`/var/cache`)

---

## สรุป
โครงสร้าง **File System Hierarchy** ใน Linux ออกแบบมาเพื่อให้แยกความรับผิดชอบของแต่ละโฟลเดอร์ได้ชัดเจน ทำให้ง่ายต่อการจัดการไฟล์ ระบบ และการพัฒนาแอปพลิเคชัน!
