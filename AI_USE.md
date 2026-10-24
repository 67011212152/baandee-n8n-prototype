# AI Usage Disclosure (AI_USE.md)

เอกสารนี้จัดทำขึ้นเพื่อชี้แจงการใช้งาน Generative AI ในโปรเจกต์ระบบประมวลผลและคัดกรองข้อร้องเรียนลูกค้าอัตโนมัติ (Automated Complaint Processing Data Product) 

---

## 1. เครื่องมือ AI ที่ใช้งาน (AI Tools Used)
* **Gemini (Google AI Assistant):** ใช้เป็นผู้ช่วยในการออกแบบ สรุปแนวคิด ตรวจสอบ และพัฒนาโค้ด JavaScript รวมถึงการแก้ปัญหาในการเชื่อมต่อ n8n Workflow

* **Code no.1 (Validate Order ID)** :
for (const item of $input.all()) {
  const orderId = String(item.json.Order_ID ?? '').trim();
  if (orderId !== '' && !orderId.toUpperCase().includes('UNKNOWN')) {
    item.json.is_order_valid = true;
  } else {
    item.json.is_order_valid = false;
  }
}
return $input.all();

* **Code no.2 (Categorize Complaint)** :
for (const item of $input.all()) {
  const msg = item.json.Message ? item.json.Message.toLowerCase() : "";

  // 1. ตั้งค่าเริ่มต้น (หมวดทั่วไป)
  let category = "General Inquiry";
  let priority = "Low";
  let team = "Customer Service";
  let sla = 48;
  let autoReply = "ขอบคุณที่ติดต่อสอบถามข้อมูล แอดมินกำลังตรวจสอบและจะรีบตอบกลับโดยเร็วที่สุดครับ";

  // 2. ดักจับคีย์เวิร์ดเพื่อแยกหมวดและกำหนดข้อความตอบกลับ
  if (msg.includes("ช้า") || msg.includes("ตาม") || msg.includes("ยังไม่ได้") || msg.includes("not arrived") || msg.includes("where is") || msg.includes("delivery status")) {
    category = "Late Delivery";
    priority = "High";
    team = "Logistics";
    sla = 24;
    autoReply = "ขออภัยในความล่าช้าครับ แผนกขนส่งกำลังเร่งตรวจสอบสถานะพัสดุและจะอัปเดตให้ทราบโดยเร็วที่สุด";
  } 
  else if (msg.includes("พัง") || msg.includes("แตก") || msg.includes("ตำหนิ") || msg.includes("ผิดสี") || msg.includes("ไม่ตรงปก") || msg.includes("defect") || msg.includes("broken")) {
    category = "Product Defect";
    priority = "High";
    team = "QC & Claims";
    sla = 48;
    autoReply = "ขออภัยในความไม่สะดวกครับ รบกวนส่งรูปภาพสินค้าที่มีปัญหาเพื่อให้ทีม QC ตรวจสอบและดำเนินการเคลมสินค้าให้ครับ";
  } 
  else if (msg.includes("ไม่ครบ") || msg.includes("ขาด") || msg.includes("หาย")) {
    category = "Missing Item";
    priority = "High";
    team = "Warehouse";
    sla = 48;
    autoReply = "ขออภัยครับ ทีมคลังสินค้ากำลังตรวจสอบกล้องวงจรปิดและรายการแพ็กสินค้าเพื่อจัดส่งชิ้นที่ตกหล่นให้ทันทีครับ";
  } 
  else if (msg.includes("เงิน") || msg.includes("คืน") || msg.includes("ใบเสร็จ") || msg.includes("โกง") || msg.includes("refund") || msg.includes("charge")) {
    category = "Refund/Billing";
    priority = "Critical";
    team = "Finance";
    sla = 12;
    autoReply = "ได้รับเรื่องคำร้องขอคืนเงินเรียบร้อยครับ ทีมการเงินกำลังตรวจสอบยอดและหลักฐานการชำระเงินให้อย่างเร่งด่วน";
  } 
  else if (msg.includes("มารยาท") || msg.includes("แย่") || msg.includes("พูดไม่ดี") || msg.includes("rude")) {
    category = "Rude Staff";
    priority = "Medium";
    team = "Customer Service";
    sla = 48;
    autoReply = "ทางผู้บริหารต้องขออภัยเป็นอย่างยิ่งกับเหตุการณ์ที่เกิดขึ้น เราจะนำไปปรับปรุงและอบรมพนักงานทันทีครับ";
  }

  // ดักจับคำรุนแรง ดัน Priority ขึ้นสูงสุด (Critical)
  if (msg.includes("ฟ้อง") || msg.includes("สคบ") || msg.includes("ประจาน") || msg.includes("แจ้งความ") || msg.includes("ระเบิด")) {
    priority = "Critical";
    sla = 12;
    autoReply = "ทางเราได้รับเรื่องร้องเรียนระดับเร่งด่วนแล้ว ผู้จัดการฝ่ายดูแลลูกค้ากำลังติดต่อกลับเพื่อเร่งแก้ไขปัญหาให้ท่านทันทีครับ";
  }

  // 3. ส่งตัวแปรทั้งหมดออกไปให้โหนด Google Sheets ใช้งานต่อ
  item.json.summary = msg.length > 50 ? msg.substring(0, 50) + "..." : msg;
  item.json.category = category;
  item.json.priority = priority;
  item.json.team = team;
  item.json.sla = sla;
  item.json.auto_reply = autoReply;
}

return $input.all();

---

## 2. ส่วนงานที่ใช้ AI ช่วยเหลือ (AI-Assisted Tasks)

1. **การเขียนและปรับแต่ง โค้ด JavaScript (JavaScript Node in n8n):**
   * ให้ AIช่วยร่างและปรับแต่งโค้ด Logic ในการดักจับคีย์เวิร์ด (Keyword Matching) เพื่อคัดแยกหมวดหมู่ปัญหา (`Category`), กำหนดระดับความเร่งด่วน (`Priority`), ทีมรับผิดชอบ (`Responsible Team`), เวลาดำเนินการ (`SLA_Hours`), และข้อความตอบกลับอัตโนมัติ (`Auto_Reply_Template`)
   * ให้ AI ช่วยแก้ไขการดึงค่าตัวแปร `auto_reply` เพื่อไม่ให้ข้อความตอบกลับเกิดการบันทึกซ้ำกันทุกแถว

2. **การออกแบบสถาปัตยกรรมระบบ (Workflow Architecture & Error Handling):**
   * ให้ AI ช่วยวิเคราะห์ปัญหา Rate Limit และ Service Unavailable เมื่อมีการส่งข้อมูลจำนวนมากพร้อมกัน
   * ให้ AI ช่วยเสนอแนวทางแก้ไขโดยเปลี่ยนมาใช้การประมวลผลด้วย JavaScript Code Node ล้วนร่วมกับ Data Validation แทนการส่งข้อความทั้งหมดไปประมวลผลบน LLM เพื่อความเสถียรและรวดเร็ว

3. **การจัดเตรียมเอกสารและ Test Cases:**
   * ให้ AI ช่วยสรุปสถิติการจัดกลุ่มข้อมูล (Aggregation) และจัดเรียงลำดับจากมากไปน้อย (Descending) เพื่อใช้ในการตรวจสอบผลลัพธ์
   * ให้ AI ช่วยเสนอและร่างเคสทดสอบ (Test Cases) และโครงสร้างเอกสารประกอบการส่งงาน

---

## 3. วิธีการตรวจสอบและทบทวนผลลัพธ์โดยทีม (Human Verification & Quality Control)

สมาชิกในทีมได้ทำการตรวจสอบผลลัพธ์ที่สร้างโดย AI ผ่านกระบวนการดังนี้:

* **Code & Logic Audit:** ตรวจสอบความถูกต้องของโค้ด JavaScript ที่ AI สร้างขึ้น โดยการทดสอบรัน (Execute Node) ใน n8n และเช็กตัวแปร Output ในแต่ละขั้นตอน
* **Data Validation in Google Sheets:** ตรวจสอบข้อมูลดิบเทียบกับข้อมูลที่บันทึกลงในแท็บ `Processed` และ `Manual Review` เพื่อยืนยันว่าไม่มีข้อมูลผิดพลาด, ค่าว่างเปล่า (Null), หรือสูตร Error
* **End-to-End Workflow Testing:** ทดลองส่งข้อมูลร้องเรียนหลากหลายรูปแบบ ทั้งเคสปกติ เคสเร่งด่วน เคสภาษาไทย/อังกฤษ และเคสที่มี `Order_ID` ผิดปกติ เพื่อยืนยันว่าระบบทำงานได้ตรงตาม Expected Result 100%

---

## 4. ส่วนงานที่สมาชิกเป็นผู้ตัดสินใจเอง (Human Decisions)

1. **การกำหนดเงื่อนไขทางธุรกิจ (Business Rules & Policy Mapping):** 
   * การเลือกหมวดหมู่ปัญหาหลัก 7 หมวด, การกำหนด Priority, SLA, และข้อความ Auto Reply ที่เหมาะสมกับธุรกิจ BaanDee Living สมาชิกเป็นผู้กำหนดเองทั้งหมด
2. **การตัดสินใจเรื่องสถาปัตยกรรมระบบ (System Architecture Decision):**
   * การตัดสินใจเลือกใช้ **Code-Based Logic Node** เป็นระบบหลักแทนการเรียกใช้ AI/LLM ในทุกข้อความ เพื่อหลีกเลี่ยงปัญหา Rate Limit และทำให้ระบบประมวลผลได้รวดเร็ว ถือเป็นการตัดสินใจของสมาชิกในทีม
3. **ระบบการกักข้อมูลเพื่อตรวจสอบโดยคน (Quarantine / Human-in-the-loop):**
   * การตัดสินใจแยกข้อมูลที่มี `Order_ID` เป็นค่า `UNKNOWN` หรือค่าว่าง ไปไว้ที่แท็บ `Manual Review` เพื่อให้เจ้าหน้าที่ (Human) เป็นผู้ตรวจสอบ เป็นเงื่อนไขที่ทีมกำหนดขึ้นเอง
4. **การอนุมัติและส่งมอบงาน (Final Approval):**
   * ผลลัพธ์สุดท้าย ตารางข้อมูล Test Cases และไฟล์เอกสารทั้งหมด ผ่านการตรวจสอบและอนุมัติโดยสมาชิกในทีมก่อนนำส่ง
