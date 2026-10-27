# BaanDee Living - Automated Customer Complaint Management System

## 1. ปัญหาทางธุรกิจ (Business Problem)
บริษัท **BaanDee Living** ประสบปัญหาในการจัดการข้อร้องเรียนและข้อสอบถามจากลูกค้า (Customer Complaints & Inquiries) ที่เพิ่มขึ้นอย่างรวดเร็ว ส่งผลกระทบต่อการดำเนินงานในหลายมิติ ดังนี้:

1. **ความล่าช้าในการคัดแยกเคส (High Handling Time):** เจ้าหน้าที่ Customer Service ต้องอ่านและเปลี่ยนหมวดหมู่เคสด้วยตนเองทีละรายการ ทำให้เกิดความล่าช้าและเคสตกค้างจำนวนมาก
2. **การส่งเรื่องผิดแผนก (Misrouting):** ข้อร้องเรียนไม่ถูกส่งไปยังทีมที่รับผิดชอบโดยตรง (เช่น ปัญหาพัสดุช้าไม่ถูกส่งไปทีม Logistics หรือสินค้าชำรุดไม่ถูกส่งไปทีม QC) ทำให้แก้ปัญหาไม่ทันตาม SLA (Service Level Agreement)
3. **ข้อความตอบกลับไม่มีมาตรฐานและไม่ตรงจุด (Generic Auto-Reply):** ลูกค้าได้รับข้อความตอบกลับอัตโนมัติแบบเดียวกันทั้งหมด ซึ่งไม่ตอบโจทย์ปัญหาเฉพาะหน้า สร้างความไม่พึงพอใจเพิ่มเติม
4. **ปัญหาคุณภาพข้อมูลดิบ (Data Quality Issues):** ข้อมูล Order ID ที่เข้ามามีความผิดพลาด เช่น เป็นค่าว่าง หรือระบุ UNKNOWN ทำให้ไม่สามารถติดตามสถานะสินค้าได้ทันที

---

## 2. ผู้ใช้งานหลัก (Target Users / Stakeholders)

1. **Customer Support Team / CS Agents:**
   * **บทบาท:** ผู้ดูแลเคสหลักในการตอบลูกค้าและติดตามเคสที่ได้รับการคัดแยกหมวดหมู่และกำหนด Priority มาแล้ว

2. **Department Leads / Functional Teams (Logistics, QC & Claims, Finance, Warehouse):**
   * **บทบาท:** รับผิดชอบเคสที่ถูกจัดสรรเข้าทีมโดยตรง สามารถแก้ไขปัญหาได้ตรงจุดตามเวลา SLA ที่กำหนดไว้

3. **Quality Assurance / Operations Manager:**
   * **บทบาท:** ติดตามเคสที่ตกหล่นในแท็บ Manual Review (Human-in-the-loop) เพื่อแก้ไขข้อมูล Order ID ที่ผิดปกติ และติดตามภาพรวมข้อร้องเรียนผ่าน Dashboard / Pivot Table

---

# 3. สถาปัตยกรรมระบบ (System Architecture)
ระบบทำงานในรูปแบบ End-to-End Automated Pipeline โดยใช้ **n8n** เป็นตัวขับเคลื่อนหลัก ร่วมกับ **Google Sheets** ในการเป็นฐานข้อมูลดิบและการบันทึกผลลัพธ์

```text
+-----------------------------------------------------------------+
|                 Schedule Trigger (Every 5 mins)                 |
+-----------------------------------------------------------------+
                                │
                                ▼
+-----------------------------------------------------------------+
|           Read Complaint & Read Policy (Google Sheets)          |
+-----------------------------------------------------------------+
                                │
                                ▼
+-----------------------------------------------------------------+
|               Data Quality Check (Code & If Node)               |
+-----------------------------------------------------------------+
                                │
                ┌───────────────┴───────────────┐
                ▼                               ▼
       [Invalid Order_ID]               [Valid Order_ID]
                │                               │
                ▼                               ▼
  +---------------------------+   +---------------------------+
  |    Manual Review Sheet    |   |   Code in JavaScript      |
  |    (Human-in-the-loop)    |   |  (Keyword Matching/SLA)   |
  +---------------------------+   +---------------------------+
                                                │
                                                ▼
                                  +---------------------------+
                                  |      Processed Sheet      |
                                  |      (Google Sheets)      |
                                  +---------------------------+

```
### รายละเอียดของส่วนประกอบระบบ (Node Responsibilities):
* **Schedule Trigger:** กระตุ้นการทำงานของ Workflow แบบอัตโนมัติทุกๆ 5 นาที
* **Read Complaint & Read Policy:** ดึงข้อมูลข้อร้องเรียนดิบและเงื่อนไขนโยบายบริษัทจาก Google Sheets
* **Code & If Node (Data Quality Check):** ตรวจสอบคุณภาพข้อมูลดิบ หากพบ Order ID เป็นค่าว่างหรือ UNKNOWN จะส่งเข้ากระบวนการ Human-in-the-loop บันทึกลงแท็บ Manual Review
* **Code in JavaScript (Core Processing Engine):**
  * คัดแยกประเภทปัญหา (Category) ตาม Keyword Matching
  * กำหนดระดับความเร่งด่วน (Priority: Critical, High, Medium, Low)
  * กำหนดทีมผู้รับผิดชอบ (Responsible Team) และเวลาตอบกลับ (SLA_Hours)
  * กำหนดข้อความตอบกลับอัตโนมัติ (Auto_Reply_Template) ที่เหมาะสมกับบริบทปัญหา
* **Processed (Google Sheets):** บันทึกผลลัพธ์ที่ผ่านการประมวลผลแล้วลงตารางหลัก
* 
## ขอบเขตระหว่างระบบ (System Boundaries & API)

ระบบ BaanDee Complaint Classifier ใช้ n8n เป็น Integration Middleware ซึ่งมีจุดตัดขอบเขตระหว่างระบบที่สำคัญดังนี้:

1. **Schedule Trigger (Job Schedule Boundary):** ระบบจะถูกตั้งเวลาให้ทำงานอัตโนมัติ (Automated Job) ทุกๆ 5 นาที 
2. **Google Sheets REST API (Data Store Connection):**
   * **Read Operation (GET):** n8n ยิง API ดึงข้อมูลคำร้องเรียน (`Complaint`) และนโยบาย (`Policy`) จาก Google Cloud
   * **Write Operation (POST):** n8n ยิง API บันทึกตั๋วที่ประมวลผลแล้วลงตาราง `Processed` และ `Manual Review` ตามเงื่อนไข

### การจัดการสถานการณ์เชื่อมต่อ (Error Handling)
* **ได้รับข้อมูลสำเร็จ (Success):** ดึงข้อมูลและบันทึกผ่าน Google Sheets API สำเร็จ ข้อมูลจะบันทึกลงตารางทันที
* **ไม่พบข้อมูล (Data Not Found):** หากดึงข้อมูลแล้วพบว่า `Order_ID` ขาดหาย โหนด `If` จะปัดเคสไปยัง `Manual Review` ทันทีเพื่อกักกันข้อมูล (Data Quarantine)
* **บริการไม่พร้อมใช้งาน (Service Failure):** หาก Google Sheets API ขัดข้อง n8n จะแสดง Error ภายใน Workflow และหยุดการทำงานเพื่อป้องกันข้อมูลสูญหาย
  
---

## 4. โครงสร้างฐานข้อมูล (Google Sheets Structure)
* **Complaint:** แท็บเก็บข้อมูลข้อร้องเรียนดิบจากลูกค้า (Raw Data)
* **Policy:** แท็บเก็บตารางนโยบาย SLA และมาตรฐานข้อความตอบกลับ
* **Processed:** แท็บเก็บผลลัพธ์ที่ผ่านการประมวลผลแล้ว (Categorized, Priority Assigned, Dynamic Auto Reply)
* **Manual Review:** แท็บเก็บเคสที่ข้อมูลไม่สมบูรณ์ (Order ID Missing) เพื่อให้เจ้าหน้าที่เข้ามาตรวจสอบด้วยตนเอง

---

## 5. สรุปผลการทดสอบระบบ (Test Cases)

| Test Case | Scenario / Input Message | Expected Category | Expected Priority | Expected Team | SLA | Expected Action / Result | Pass/Fail |
| :---: | :--- | :---: | :---: | :---: | :---: | :--- | :---: |
| **TC-01** | "Where is my order? Delivery is late" | Late Delivery | High | Logistics | 24h | บันทึกลง Processed + Auto Reply ขนส่ง | **PASS** |
| **TC-02** | "สินค้าพัง มีตำหนิ แตกหัก" | Product Defect | High | QC & Claims | 48h | บันทึกลง Processed + Auto Reply เคลม | **PASS** |
| **TC-03** | "ขอคืนเงิน โดนโกง คิดเงินผิด" | Refund/Billing | Critical | Finance | 12h | บันทึกลง Processed + Auto Reply การเงิน | **PASS** |
| **TC-04** | "จะไปฟ้อง สคบ. / แจ้งความ" | (คีย์เวิร์ดรุนแรง) | Critical | (ตามหมวด) | 12h | ดัน Priority เป็น Critical ทันที | **PASS** |
| **TC-05** | Order_ID = "ORD-UNKNOWN-77" | - | - | - | - | คัดแยกเข้าแท็บ **Manual Review** | **PASS** |

---

## 6. ขั้นตอนการติดตั้งและการใช้งาน (Setup & Installation Guide)

### สิ่งที่ต้องเตรียมก่อนติดตั้ง (Prerequisites):
1. บัญชี n8n (Cloud หรือ Self-hosted instance)
2. บัญชี Google Workspace ที่สามารถใช้งาน Google Sheets API ได้
3. ไฟล์ Workflow JSON (`304_project.json`) และไฟล์ Google Sheets แม่แบบ

### ขั้นตอนการติดตั้ง:
1. **การเตรียม Google Sheets:**
   * สร้าง Google Sheets ใหม่ และทำสำเนาแท็บให้มี 4 แท็บ ได้แก่ `Complaint`, `Policy`, `Processed`, และ `Manual Review`
   * ใส่ข้อมูลข้อร้องเรียนลงในแท็บ `Complaint`
2. **การนำเข้า Workflow ใน n8n:**
   * เปิด n8n ➔ สร้าง Workflow ใหม่ ➔ กดเมนูมุมขวาบนเลือก **Import from File**
   * เลือกไฟล์ `304_project.json`
3. **การตั้งค่า Credentials:**
   * เชื่อมต่อ Google Sheets OAuth2 / Service Account ในทุกโหนด Google Sheets
   * ระบุ `Document ID` ของ Google Sheets ที่สร้างไว้ในทุกโหนด
4. **การรันใช้งาน:**
   * กดปุ่ม **Execute Workflow** เพื่อทดสอบ หรือเปิดสวิตช์ **Active** เพื่อให้ระบบทำงานอัตโนมัติตามเวลา
