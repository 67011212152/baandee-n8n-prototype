สรุปผลการทดสอบระบบ (Test Cases)

| Test Case | Scenario / Input Message | Expected Category | Expected Priority | Expected Team | SLA | Expected Action / Result | Pass/Fail |
| :---: | :--- | :---: | :---: | :---: | :---: | :--- | :---: |
| **TC-01** | "Where is my order? Delivery is late" | Late Delivery | High | Logistics | 24h | บันทึกลง Processed + Auto Reply ขนส่ง | **PASS** |
| **TC-02** | "สินค้าพัง มีตำหนิ แตกหัก" | Product Defect | High | QC & Claims | 48h | บันทึกลง Processed + Auto Reply เคลม | **PASS** |
| **TC-03** | "ขอคืนเงิน โดนโกง คิดเงินผิด" | Refund/Billing | Critical | Finance | 12h | บันทึกลง Processed + Auto Reply การเงิน | **PASS** |
| **TC-04** | "จะไปฟ้อง สคบ. / แจ้งความ" | (คีย์เวิร์ดรุนแรง) | Critical | (ตามหมวด) | 12h | ดัน Priority เป็น Critical ทันที | **PASS** |
| **TC-05** | Order_ID = "ORD-UNKNOWN-77" | - | - | - | - | คัดแยกเข้าแท็บ **Manual Review** | **PASS** |
