# baandee-n8n-prototype
📦 BaanDee Living - Automated Customer Complaint Management System
📌 1. ปัญหาทางธุรกิจ (Business Problem)
บริษัท BaanDee Living ประสบปัญหาในการจัดการข้อร้องเรียนและข้อสอบถามจากลูกค้า (Customer Complaints & Inquiries) ที่เพิ่มขึ้นอย่างรวดเร็ว ส่งผลกระทบต่อการดำเนินงานในหลายมิติ ดังนี้:
1. ความล่าช้าในการคัดแยกเคส (High Handling Time): เจ้าหน้าที่ Customer Service ต้องอ่านและเปลี่ยนหมวดหมู่เคสด้วยตนเองทีละรายการ ทำให้เกิดความล่าช้าและเคสตกค้างจำนวนมาก
2. การส่งเรื่องผิดแผนก (Misrouting): ข้อร้องเรียนไม่ถูกส่งไปยังทีมที่รับผิดชอบโดยตรง (เช่น ปัญหาพัสดุช้าไม่ถูกส่งไปทีม Logistics หรือสินค้าชำรุดไม่ถูกส่งไปทีม QC) ทำให้แก้ปัญหาไม่ทันตาม SLA (Service Level Agreement)
3. ข้อความตอบกลับไม่มีมาตรฐานและไม่ตรงจุด (Generic Auto-Reply): ลูกค้าได้รับข้อความตอบกลับอัตโนมัติแบบเดียวกันทั้งหมด ซึ่งไม่ตอบโจทย์ปัญหาเฉพาะหน้า สร้างความไม่พึงพอใจเพิ่มเติม
4. ปัญหาคุณภาพข้อมูลดิบ (Data Quality Issues): ข้อมูล Order ID ที่เข้ามามีความผิดพลาด เช่น เป็นค่าว่าง หรือระบุ ⁠UNKNOWN⁠ ทำให้ไม่สามารถติดตามสถานะสินค้าได้ทันที
👤 2. ผู้ใช้งานหลัก (Target Users / Stakeholders)
1. Customer Support Team / CS Agents:
￼ บทบาท: ผู้ดูแลเคสหลักในการตอบลูกค้าและติดตามเคสที่ได้รับการคัดแยกหมวดหมู่และกำหนด Priority มาแล้ว
2. Department Leads / Functional Teams (Logistics, QC & Claims, Finance, Warehouse):
￼ บทบาท: รับผิดชอบเคสที่ถูกจัดสรรเข้าทีมโดยตรง สามารถแก้ไขปัญหาได้ตรงจุดตามเวลา SLA ที่กำหนดไว้
3. Quality Assurance / Operations Manager:
￼ บทบาท: ติดตามเคสที่ตกหล่นในแท็บ ⁠Manual Review⁠ (Human-in-the-loop) เพื่อแก้ไขข้อมูล Order ID ที่ผิดปกติ และติดตามภาพรวมข้อร้องเรียนผ่าน Dashboard / Pivot Table
🏗️ 3. สถาปัตยกรรมระบบ (System Architecture)
ระบบทำงานในรูปแบบ End-to-End Automated Pipeline โดยใช้ n8n เป็นตัวขับเคลื่อนหลัก ร่วมกับ Google Sheets ในการเป็นฐานข้อมูลดิบและการบันทึกผลลัพธ์
🧩 รายละเอียดของส่วนประกอบระบบ (Node Responsibilities):
￼ Schedule Trigger: กระตุ้นการทำงานของ Workflow แบบอัตโนมัติทุกๆ 5 นาที
￼ Read Complaint & Read Policy: ดึงข้อมูลข้อร้องเรียนดิบและเงื่อนไขนโยบายบริษัทจาก Google Sheets
￼ Code & If Node (Data Quality Check): ตรวจสอบคุณภาพข้อมูลดิบ หากพบ Order ID เป็นค่าว่างหรือ ⁠UNKNOWN⁠ จะส่งเข้ากระบวนการ Human-in-the-loop บันทึกลงแท็บ ⁠Manual Review⁠
￼ Code in JavaScript (Core Processing Engine):
￼ คัดแยกประเภทปัญหา (⁠Category⁠) ตาม Keyword Matching
￼ กำหนดระดับความเร่งด่วน (⁠Priority⁠: Critical, High, Medium, Low)
￼ กำหนดทีมผู้รับผิดชอบ (⁠Responsible Team⁠) และเวลาตอบกลับ (⁠SLA_Hours⁠)
￼ กำหนดข้อความตอบกลับอัตโนมัติ (⁠Auto_Reply_Template⁠) ที่เหมาะสมกับบริบทปัญหา
📊 4. โครงสร้างฐานข้อมูล (Google Sheets Structure)
￼ ⁠Complaint⁠: แท็บเก็บข้อมูลข้อร้องเรียนดิบจากลูกค้า (Raw Data)
￼ ⁠Policy⁠: แท็บเก็บตารางนโยบาย SLA และมาตรฐานข้อความตอบกลับ
￼ ⁠Processed⁠: แท็บเก็บผลลัพธ์ที่ผ่านการประมวลผลแล้ว (Categorized, Priority Assigned, Dynamic Auto Reply)
￼ ⁠Manual Review⁠: แท็บเก็บเคสที่ข้อมูลไม่สมบูรณ์ (Order ID Missing) เพื่อให้เจ้าหน้าที่เข้ามาตรวจสอบด้วยตนเอง
⚙️ 5. ขั้นตอนการติดตั้งและการใช้งาน (Setup & Installation Guide)
สิ่งที่ต้องเตรียมก่อนติดตั้ง (Prerequisites):
1. บัญชี n8n (Cloud หรือ Self-hosted instance)
2. บัญชี Google Workspace ที่สามารถใช้งาน Google Sheets API ได้
3. ไฟล์ Workflow JSON (⁠304_project.json⁠) และไฟล์ Google Sheets แม่แบบ
ขั้นตอนการติดตั้ง:
1. การเตรียม Google Sheets:
￼ สร้าง Google Sheets ใหม่ และทำสำเนาแท็บให้มี 4 แท็บ ได้แก่ ⁠Complaint⁠, ⁠Policy⁠, ⁠Processed⁠, และ ⁠Manual Review⁠
￼ ใส่ข้อมูลข้อร้องเรียนลงในแท็บ ⁠Complaint⁠
2. การนำเข้า Workflow เข้าสู่ n8n:
￼ เปิดหน้าต่าง n8n เลือก Workflows > Import from File
￼ เลือกไฟล์ ⁠304_project.json⁠
￼ ตั้งค่า Google Sheets OAuth2 Credentials ให้เชื่อมต่อกับบัญชี Google ของคุณ
3. การเชื่อมโยง Google Sheets กับ n8n Nodes:
￼ เปิดโหนด ⁠Read Complaint⁠, ⁠Read Policy⁠, ⁠Processed⁠, และ ⁠Manual Review⁠
￼ เลือก Document ID ให้ตรงกับไฟล์ Google Sheets ที่เตรียมไว้
￼ ตรวจสอบ Sheet Name ให้ตรงกับชื่อแท็บใน Google Sheets
4. การทดสอบและเปิดใช้งานระบบ:
￼ กดปุ่ม Execute Workflow เพื่อทดสอบการทำงาน
￼ สับสวิตช์ Active เป็น ⁠ON⁠ เพื่อให้ระบบทำงานอัตโนมัติรอบตามเวลาที่ตั้งไว้
🧪 6. การทดสอบระบบ (Test Cases Verification)
ระบบได้รับการทดสอบผ่าน Test Cases หลัก 5 กรณี เพื่อยืนยันความถูกต้องและความเสถียร:
ID
กรณีการทดสอบ (Test Case Description)
Input Message / Condition
Expected Result
Status
TC01
ติดตามพัสดุช้า (Late Delivery)
"Where is my order? ส่งช้ามาก"
Category: Late Delivery, Team: Logistics, SLA: 24h
PASS
TC02
สินค้าชำรุดเสียหาย (Product Defect)
"สินค้าแตกหัก พังใช้งานไม่ได้"
Category: Product Defect, Team: QC & Claims, SLA: 48h
PASS
TC03
คำร้องขอคืนเงิน (Refund / Billing)
"ขอเงินคืนด้วย โอนเงินไปแล้ว"
Category: Refund/Billing, Priority: Critical, SLA: 12h
PASS
TC04
ดักจับคำรุนแรง/ขู่ฟ้อง (Escalation)
"ส่งของช้า จะแจ้ง สคบ ประจาน"
Priority: Critical, SLA: 12h (Escalated)
PASS
TC05
ข้อมูล Order ID ผิดปกติ (Data Quality Failure)
Order_ID = UNKNOWN หรือค่าว่าง
คัดแยกเคสไปที่แท็บ Manual Review
PASS

👥 จัดทำโดย
￼ โปรเจกต์วิชา: 304_project (Automated Customer Complaint Management)
￼ ระบบประมวลผล: n8n Workflow Automation Engine & Google Sheets API
