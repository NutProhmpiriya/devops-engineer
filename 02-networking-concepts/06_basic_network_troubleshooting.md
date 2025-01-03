การแก้ไขปัญหาเครือข่ายพื้นฐานเป็นทักษะสำคัญที่ช่วยให้ระบุปัญหาและแก้ไขข้อผิดพลาดในระบบเครือข่ายได้รวดเร็ว โดยเครื่องมือที่นิยมใช้ได้แก่ **ping**, **traceroute**, และ **netstat** ซึ่งแต่ละตัวมีความสามารถเฉพาะตัวและวิธีการใช้งานที่เหมาะสม

---

```markdow
# Basic Network Troubleshooting

## 1. เครื่องมือแก้ไขปัญหาเครือข่ายพื้นฐาน

### 1.1 **ping**
`ping` ใช้สำหรับตรวจสอบว่าปลายทาง (Host) สามารถเข้าถึงได้หรือไม่ และวัดความหน่วง (Latency) ในการสื่อสารระหว่างเครื่องต้นทางและปลายทาง

#### **คำสั่งพื้นฐาน**
```bash
ping [hostname or IP address]
```

#### **ตัวอย่าง**
```bash
ping google.com
```

#### **ผลลัพธ์ที่ได้**
- **Reply from**: หมายความว่าปลายทางสามารถตอบกลับได้
- **Request timed out**: ปลายทางไม่ตอบสนอง อาจมีปัญหาเครือข่าย
- **Packet Loss**: ใช้ระบุคุณภาพของการเชื่อมต่อ

#### **ตัวเลือกเพิ่มเติม**
- `-c [count]`: ระบุจำนวนครั้งที่ส่ง Ping (Linux/Mac)
  ```bash
  ping -c 5 google.com
  ```
- `-t`: ส่ง Ping แบบต่อเนื่อง (Windows)
  ```bash
  ping -t google.com
  ```

---

### 1.2 **traceroute/tracert**
`traceroute` (Linux/Mac) หรือ `tracert` (Windows) ใช้เพื่อตรวจสอบเส้นทางที่แพ็กเก็ตเดินทางไปถึงปลายทาง และระบุปัญหาหากการเชื่อมต่อล้มเหลว

#### **คำสั่งพื้นฐาน**
- **Linux/Mac**:
  ```bash
  traceroute [hostname or IP address]
  ```
- **Windows**:
  ```bash
  tracert [hostname or IP address]
  ```

#### **ตัวอย่าง**
```bash
traceroute google.com
```

#### **ผลลัพธ์ที่ได้**
- แสดงรายการของ Router ที่แพ็กเก็ตผ่าน
- ระบุค่าความหน่วง (Latency) ในแต่ละ Hop
- ถ้าหยุดที่บาง Hop และไม่ไปต่อ อาจหมายถึงปัญหาในเส้นทางเครือข่าย

#### **ตัวเลือกเพิ่มเติม**
- `-n`: แสดง IP Address แทนชื่อ DNS (Linux/Mac)
  ```bash
  traceroute -n google.com
  ```

---

### 1.3 **netstat**
`netstat` ใช้เพื่อตรวจสอบสถานะการเชื่อมต่อเครือข่าย, พอร์ตที่กำลังใช้งาน, และข้อมูลเกี่ยวกับ TCP/UDP บนเครื่อง

#### **คำสั่งพื้นฐาน**
```bash
netstat
```

#### **ตัวเลือกเพิ่มเติม**
- `-a`: แสดงการเชื่อมต่อทั้งหมด (ทั้งที่กำลังใช้งานและรอการเชื่อมต่อ)
  ```bash
  netstat -a
  ```
- `-n`: แสดงหมายเลข IP และพอร์ตในรูปแบบตัวเลข (ไม่แปลงชื่อ)
  ```bash
  netstat -an
  ```
- `-t` (Linux): แสดงเฉพาะการเชื่อมต่อแบบ TCP
  ```bash
  netstat -t
  ```

#### **ตัวอย่างผลลัพธ์**
| Proto  | Local Address       | Foreign Address     | State       |
|--------|---------------------|---------------------|-------------|
| TCP    | 192.168.1.5:5000    | 93.184.216.34:443   | ESTABLISHED |

- **Proto**: ประเภทโปรโตคอล (TCP/UDP)
- **Local Address**: IP Address และพอร์ตในเครื่องต้นทาง
- **Foreign Address**: ปลายทางที่กำลังเชื่อมต่อ
- **State**: สถานะ เช่น `ESTABLISHED`, `LISTENING`, `CLOSED`

---

## 2. การวิเคราะห์และแก้ไขปัญหาด้วยเครื่องมือเหล่านี้

### **ปัญหา: ไม่สามารถเข้าถึงเว็บไซต์**
1. ใช้ `ping` ตรวจสอบว่าปลายทางตอบสนองหรือไม่:
   ```bash
   ping google.com
   ```
   - หากไม่มีการตอบกลับ อาจเกิดจากปัญหา DNS, Firewall, หรือเส้นทางเครือข่าย

2. ใช้ `traceroute` เพื่อตรวจสอบเส้นทาง:
   ```bash
   traceroute google.com
   ```
   - ระบุว่าแพ็กเก็ตหยุดที่ Hop ใด และวิเคราะห์ปัญหาในเส้นทางนั้น

3. ตรวจสอบการเชื่อมต่อเครือข่ายบนเครื่องด้วย `netstat`:
   ```bash
   netstat -an
   ```
   - ดูว่ามีการเปิดพอร์ตที่เกี่ยวข้อง เช่น `80` หรือ `443`

---

### **ปัญหา: ความเร็วอินเทอร์เน็ตช้าหรือมี Packet Loss**
1. ใช้ `ping` กับปลายทางที่เชื่อถือได้:
   ```bash
   ping 8.8.8.8
   ```
   - หากมี Packet Loss อาจหมายถึงปัญหาเครือข่ายระหว่างเครื่องต้นทางและปลายทาง

2. ใช้ `traceroute` เพื่อตรวจสอบว่าเส้นทางมีปัญหาที่ Hop ใด:
   ```bash
   traceroute 8.8.8.8
   ```

---

### **ปัญหา: พอร์ตถูกบล็อกหรือไม่**
1. ใช้ `netstat` เพื่อตรวจสอบว่าพอร์ตเปิดอยู่หรือไม่:
   ```bash
   netstat -an | grep LISTEN
   ```

2. ใช้ `ping` ตรวจสอบการเชื่อมต่อทั่วไป และ `telnet` ตรวจสอบพอร์ตเฉพาะ:
   ```bash
   telnet [IP Address] [Port]
   ```
   - หากเชื่อมต่อไม่ได้ อาจมี Firewall หรือ Security Group บล็อกพอร์ตนั้น

---

## 3. สรุปการใช้งาน
| เครื่องมือ       | วัตถุประสงค์                                 | คำสั่งตัวอย่าง                     |
|-------------------|-----------------------------------------------|-------------------------------------|
| **ping**          | ตรวจสอบการตอบสนองของปลายทาง                 | `ping google.com`                  |
| **traceroute**    | ตรวจสอบเส้นทางและ Hop ที่แพ็กเก็ตผ่าน         | `traceroute google.com`            |
| **netstat**       | ตรวจสอบการเชื่อมต่อเครือข่ายและสถานะของพอร์ต | `netstat -an`                      |

