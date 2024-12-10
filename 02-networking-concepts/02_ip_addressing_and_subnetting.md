# IP Addressing and Subnetting

## 1. IP Addressing
IP Address (Internet Protocol Address) คือหมายเลขที่ใช้ระบุอุปกรณ์ในเครือข่าย เช่น คอมพิวเตอร์, เซิร์ฟเวอร์, หรือเราเตอร์ เพื่อให้สามารถสื่อสารกันได้ มี 2 รูปแบบ:
- **IPv4 (Internet Protocol Version 4):** ใช้กันอย่างแพร่หลายในปัจจุบัน
- **IPv6 (Internet Protocol Version 6):** ออกแบบมาเพื่อแก้ปัญหาการขาดแคลน IP Address ใน IPv4

---

## 2. IPv4 (รายละเอียดเพิ่มเติม)
IPv4 มีความยาว 32 บิต แบ่งเป็น 4 ส่วน (Octet) แต่ละส่วนมีค่า 0-255 เช่น `192.168.0.1`

### 2.1 Classes in IPv4
- **Class A:** ใช้สำหรับเครือข่ายขนาดใหญ่ (IP: 1.0.0.0 - 126.255.255.255)
  - Default Subnet Mask: `255.0.0.0`
  - ใช้ IP ได้: ประมาณ 16 ล้านเครื่อง
- **Class B:** สำหรับเครือข่ายขนาดกลาง (IP: 128.0.0.0 - 191.255.255.255)
  - Default Subnet Mask: `255.255.0.0`
  - ใช้ IP ได้: ประมาณ 65,000 เครื่อง
- **Class C:** สำหรับเครือข่ายขนาดเล็ก (IP: 192.0.0.0 - 223.255.255.255)
  - Default Subnet Mask: `255.255.255.0`
  - ใช้ IP ได้: 254 เครื่อง
- **Class D:** ใช้สำหรับ Multicast (IP: 224.0.0.0 - 239.255.255.255)
- **Class E:** ใช้สำหรับวิจัย (IP: 240.0.0.0 - 255.255.255.255)

### 2.2 Subnetting in IPv4
Subnetting ใช้สำหรับแบ่งเครือข่ายใหญ่เป็นเครือข่ายย่อย
- **Subnet Mask:** กำหนดขอบเขต เช่น `255.255.255.0`
- **CIDR (Classless Inter-Domain Routing):** เช่น `/24` หมายถึง Subnet Mask `255.255.255.0`
- **วิธีคำนวณจำนวน IP Address:**

จำนวน IP Address = 2^(32 - Subnet Mask)


#### ตัวอย่าง Subnet Mask
| CIDR Notation | Subnet Mask     | จำนวน IP Address |
|---------------|-----------------|-------------------|
| /24           | 255.255.255.0  | 256 (ลด 2 = 254)  |
| /25           | 255.255.255.128| 128 (ลด 2 = 126)  |
| /26           | 255.255.255.192| 64  (ลด 2 = 62)   |

---

## 3. IPv6 (รายละเอียดเพิ่มเติม)
IPv6 ใช้ 128 บิต แบ่งเป็น 8 ส่วน (Hextet) เช่น `2001:0db8:85a3:0000:0000:8a2e:0370:7334`

### 3.1 IPv6 Address Types
1. **Unicast:** ใช้สื่อสารกับอุปกรณ์เดียว
2. **Multicast:** ส่งข้อมูลถึงกลุ่มอุปกรณ์
3. **Anycast:** ใช้ส่งข้อมูลไปยังอุปกรณ์ที่อยู่ใกล้ที่สุดในเครือข่าย

### 3.2 IPv6 Subnetting
IPv6 ใช้ Prefix Length เช่น `/64`
- **วิธีการแบ่งเครือข่าย:** ใช้ Prefix เช่น `/48`, `/56`, `/64` เพื่อกำหนดขนาดเครือข่าย
- **ตัวอย่าง:**  

IPv6 Address: 2001:db8:abcd::/64 Prefix: 64 (64 บิตแรกใช้ระบุ Network)


---

## 4. วิธีตรวจสอบ IP Address
### 4.1 ตรวจสอบด้วย Command Line (CLI)
#### Windows:
- ดู IP Address: `ipconfig`
- ตรวจสอบ Default Gateway: `tracert`

#### Linux/Mac:
- ดู IP Address: `ip addr` หรือ `ifconfig`
- ตรวจสอบ Routing Table: `ip route` หรือ `netstat -rn`

### 4.2 ตรวจสอบออนไลน์
- ใช้เว็บไซต์ เช่น `https://whatismyipaddress.com`

### 4.3 ตรวจสอบด้วย Tools อื่น ๆ
- **Wireshark:** สำหรับวิเคราะห์ Packet
- **Nmap:** สำหรับสแกน IP และ Port

---

## 5. สรุปเปรียบเทียบ IPv4 และ IPv6
| Feature                | IPv4                  | IPv6                      |
|------------------------|-----------------------|---------------------------|
| Address Length         | 32 bits              | 128 bits                 |
| Example Address        | 192.168.1.1          | 2001:db8::1              |
| Subnetting Format      | CIDR (e.g., /24)     | Prefix (e.g., /64)       |
| Broadcast Address      | Yes                  | No                       |
| NAT (Network Address Translation) | ใช้บ่อย | ไม่จำเป็น (IPv6 มี IP เพียงพอ) |

---

