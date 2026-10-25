# baandee-n8n-prototype
# Customer Complaint Automation & Categorization System (BaanDee Living)

ระบบประมวลผล คัดแยกหมวดหมู่ และจัดการข้อร้องเรียนของลูกค้าอัตโนมัติ (Automated Complaint Handling System) พัฒนาขึ้นด้วย **n8n** และ **Google Sheets API** เพื่อช่วยเพิ่มประสิทธิภาพการทำงานของทีมสนับสนุนลูกค้า (Customer Service) และทีมปฏิบัติการภายในองค์กร

---

## 1. ปัญหาทางธุรกิจ (Business Problem)
ในปัจจุบัน บริษัท BaanDee Living ได้รับข้อร้องเรียนและสอบถามจากลูกค้าผ่านช่องทางดิจิทัลเป็นจำนวนมาก โดยพบปัญหาหลักในการจัดการข้อมูลดังนี้:
* **ปริมาณข้อความสูงและล่าช้าในการตอบกลับ:** การคัดแยกประเภทปัญหาและส่งต่อให้ทีมที่เกี่ยวข้องด้วยมนุษย์ (Manual) ใช้เวลานาน ทำให้ไม่ทันกำหนดเวลา SLA
* **ข้อมูลดิบขาดคุณภาพ (Data Quality Issues):** พบปัญหา `Order_ID` สูญหาย หรือระบุเป็น `UNKNOWN` ซึ่งหากส่งข้อความตอบกลับอัตโนมัติทันทีอาจเกิดความผิดพลาดในการสื่อสาร
* **ขาดการจัดลำดับความสำคัญ (Priority Handling):** ข้อร้องเรียนเร่งด่วน (เช่น การขอคืนเงิน หรือข้อความขู่ฟ้องร้อง) ไม่ถูกดันขึ้นมาจัดการก่อน ทำให้เกิดความเสี่ยงทางธุรกิจ

---

## 2. ผู้ใช้งานหลัก (Target Users / Stakeholders)
1. **Customer Support Team (ทีมบริการลูกค้า):** ใช้ติดตามเคสข้อร้องเรียนทั่วไปและส่ง Auto Reply ให้ลูกค้าอย่างรวดเร็ว
2. **Specialized Teams (ทีมปฏิบัติการเฉพาะทาง):** 
   * **Logistics:** ดูแลเคสส่งสินค้าล่าช้า (`Late Delivery`)
   * **QC & Claims:** ดูแลเคสสินค้าชำรุด/แตกหัก (`Product Defect`)
   * **Warehouse:** ดูแลเคสสินค้าขาดหาย (`Missing Item`)
   * **Finance:** ดูแลเคสขอคืนเงินและใบเสร็จ (`Refund/Billing`)
3. **Operations Manager / Admin:** เข้ามาตรวจสอบเคสที่ต้องใช้มนุษย์ตัดสินใจเพิ่มเติมผ่านแท็บ `Manual Review`

---

## 3. สถาปัตยกรรมระบบ (System Architecture)

ระบบทำงานในรูปแบบ **Event-Driven & Rule-Based Automation Workflow** บน n8n ดังนี้:
[Schedule Trigger (Every 5 mins)]
          │
          ▼
[Read Google Sheets: Complaint & Policy]
          │
          ▼
[Data Validation & Quarantine Check (Code & If Node)]
       ├───► (Order_ID Invalid / UNKNOWN) ───► [Write to "Manual Review" Sheet] (Human-in-the-loop)
       │
       └───► (Order_ID Valid)
                 │
                 ▼
      [JavaScript Processing Node]
        - Keyword Matching & Rules
        - Category & Team Assignment
        - Priority & SLA Calculation
        - Dynamic Auto Reply Selection
                 │
                 ▼
      [Write to "Processed" Sheet]


