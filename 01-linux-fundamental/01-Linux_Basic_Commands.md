
# คำสั่งพื้นฐานใน Linux

## การจัดการไฟล์และโฟลเดอร์
1. **ls**  
   แสดงรายการไฟล์และโฟลเดอร์ในโฟลเดอร์ปัจจุบัน  
   ```bash
   ls
   ls -l
   ls -a
   ```

2. **cd**  
   เปลี่ยนโฟลเดอร์ที่ใช้งาน  
   ```bash
   cd /path/to/directory
   cd ..
   cd ~
   ```

3. **pwd**  
   แสดงตำแหน่งโฟลเดอร์ปัจจุบัน  
   ```bash
   pwd
   ```

4. **mkdir**  
   สร้างโฟลเดอร์ใหม่  
   ```bash
   mkdir my_folder
   ```

5. **rm**  
   ลบไฟล์หรือโฟลเดอร์  
   ```bash
   rm file.txt
   rm -r folder_name
   ```

6. **cp**  
   คัดลอกไฟล์หรือโฟลเดอร์  
   ```bash
   cp file1.txt file2.txt
   cp -r folder1/ folder2/
   ```

7. **mv**  
   ย้ายหรือเปลี่ยนชื่อไฟล์/โฟลเดอร์  
   ```bash
   mv old_name.txt new_name.txt
   mv file.txt /path/to/new_location/
   ```

## การจัดการข้อความ
8. **cat**  
   แสดงเนื้อหาของไฟล์  
   ```bash
   cat file.txt
   ```

9. **nano**  
   แก้ไขไฟล์ข้อความ (ตัวแก้ไขแบบง่าย)  
   ```bash
   nano file.txt
   ```

10. **grep**  
    ค้นหาข้อความในไฟล์  
    ```bash
    grep "keyword" file.txt
    ```

## การจัดการระบบ
11. **sudo**  
    รันคำสั่งในฐานะผู้ดูแลระบบ  
    ```bash
    sudo apt update
    ```

12. **top**  
    แสดงกระบวนการที่กำลังรันอยู่ในระบบแบบเรียลไทม์  
    ```bash
    top
    ```

13. **ps**  
    แสดงกระบวนการที่กำลังทำงาน  
    ```bash
    ps aux
    ```

14. **kill**  
    ยุติกระบวนการที่ทำงาน  
    ```bash
    kill -9 PID
    ```

## อื่น ๆ
15. **man**  
    อ่านคู่มือการใช้งานคำสั่ง  
    ```bash
    man ls
    ```

16. **chmod**  
    เปลี่ยนสิทธิ์การเข้าถึงไฟล์/โฟลเดอร์  
    ```bash
    chmod 755 file.sh
    ```

17. **df**  
    แสดงพื้นที่ดิสก์ที่เหลืออยู่  
    ```bash
    df -h
    ```

18. **du**  
    แสดงขนาดของโฟลเดอร์/ไฟล์  
    ```bash
    du -sh folder_name
    ```

19. **tar**  
    บีบอัด/แตกไฟล์ที่ถูกบีบอัด  
    ```bash
    tar -cvf archive.tar folder/
    tar -xvf archive.tar
    ```

20. **wget**  
    ดาวน์โหลดไฟล์จากอินเทอร์เน็ต  
    ```bash
    wget http://example.com/file.zip
    ```
