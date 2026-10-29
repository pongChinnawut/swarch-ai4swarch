**ภาษา:** ภาษาไทย | [English](bonus-recover-decision.en.md)

# โบนัส (ไม่บังคับ ทำที่บ้าน): กู้คืนการตัดสินใจที่ไม่ได้บันทึกไว้ (ประมาณ 15 นาที)

> **เชื่อมโยงกับการบรรยาย:** T5 (Dhar et al. สร้างการตัดสินใจจากโค้ดและแผนภาพเพื่อลด *การสูญหายของความรู้* หรือ knowledge vaporization), AICH4
> **คำถาม:** เมื่อ AI อธิบายว่า*ทำไม*ระบบจึงถูกสร้างแบบนี้ คำอธิบายส่วนใดมาจากหลักฐาน และส่วนใดเป็นการเดาอย่างมั่นใจ

## ลงมือทำ

เริ่มแชตหรือ session ใหม่ แล้วส่งข้อความนี้ให้ agent

```
Follow prompts/bonus-recover-decision.md
decision: Why do all services communicate synchronously over gRPC, using one shared demo.proto?
evidence: system/online-boutique/protos/demo.proto, system/online-boutique/src/checkoutservice/main.go, system/online-boutique/src/frontend/main.go
language: Thai
```

บรรทัด `language: Thai` ทำให้ ADR ที่กู้คืนเป็นภาษาไทย หากต้องการภาษาอังกฤษให้ลบบรรทัดนี้

agent จะเขียน ADR ที่กู้คืนมา พร้อมตาราง **Claim check** ไว้ที่ [`outputs/bonus-recovered-adr.md`](outputs/bonus-recovered-adr.md)

## ตรวจสอบตาราง claim check

เลือกข้อกล่าวอ้าง**ห้าข้อ**จากตารางของ agent โดยเป็นข้อที่ agent ระบุว่า *Evidenced* (มีหลักฐาน) อย่างน้อยสองข้อ และ *Inferred* (อนุมานเอา) อย่างน้อยสองข้อ แล้วตรวจสอบแต่ละข้อด้วยตัวเอง

| # | ข้อกล่าวอ้าง (ย่อ) | AI ระบุว่า | นิสิตระบุว่า | เหตุผล (path ของไฟล์หรือการให้เหตุผล) |
|---|---|---|---|---|
| 1 | demo.proto รวมสัญญา unary RPC ของเก้าบริการ (Claim 1) | Evidenced | Evidenced | system/online-boutique/protos/demo.proto:15–244 ประกาศเก้าบริการด้วย service และ rpc โดยไม่มี stream แต่ยังไม่พิสูจน์ว่าผู้เรียกทุกตัวทำงานแบบรอผล |
| 2 | หลายสัญญาใช้ Money ร่วมกัน (Claim 2) | Evidenced | Evidenced | system/online-boutique/protos/demo.proto:82,146–161,169,189 มี Product.price_usd, CurrencyConversionRequest.from และ ChargeRequest.amount ซึ่งใช้ชนิด Money เดียวกัน |
| 3 | ความเข้าใจง่ายของ workflow อาจเป็นแรงจูงใจในการออกแบบ (Claim 6) | Inferred | Inferred | system/online-boutique/src/checkoutservice/main.go:230–279 (PlaceOrder) แสดงลำดับการทำงาน แต่ไม่มีข้อความว่าผู้ออกแบบเลือกวิธีนี้เพื่อให้เข้าใจง่าย |
| 4 | ความสอดคล้องของข้อมูลอาจเป็นแรงจูงใจให้รวม schema (Claim 7) | Inferred | Inferred | system/online-boutique/protos/demo.proto:82,146–161,169,189 แสดงการใช้ Money ร่วมกัน แต่ไม่ได้บันทึกเหตุผลที่รวมสัญญาไว้ในไฟล์เดียว |
| 5 | error จากชำระเงินและจัดส่งทำให้ตอบ error แต่ตะกร้าและอีเมลจัดการต่างกัน (Claim 17) | Evidenced | Evidenced | system/online-boutique/src/checkoutservice/main.go:252–279 (PlaceOrder) คืน error เมื่อ chargeCard หรือ shipOrder ล้มเหลว แต่ละทิ้ง error จาก emptyUserCart และบันทึกคำเตือนจาก sendOrderConfirmation ก่อนตอบสำเร็จ |

**นับ:** จากข้อกล่าวอ้างทั้งหมด agent ระบุว่า *Evidenced* กี่ข้อ **6 ข้อ** ตามที่นิสิตตรวจ **6 ข้อ** (Claim 1–5 และ 17)

## สะท้อนคิด

1. agent เคยระบุการเดาว่าเป็น *Evidenced* หรือไม่ ยกตัวอย่าง หรืออธิบายว่าตรวจสอบอย่างไรว่าไม่มี

   **คำตอบ:** ไม่พบใน ADR ฉบับนี้ เมื่อตรวจข้อที่ระบุว่า Evidenced ทั้งหกข้อ (Claim 1–5 และ 17) เทียบกับสัญลักษณ์และบรรทัดที่อ้าง พบว่าเป็นโครงสร้างสัญญาหรือพฤติกรรมที่เห็นจากโค้ด ส่วนแรงจูงใจ เช่น “เลือกเพื่อให้ workflow เข้าใจง่าย” ถูกระบุเป็น Inferred ทั้งนี้ unary RPC เพียงอย่างเดียวไม่พิสูจน์ว่าผู้เรียกทุกตัวทำงานแบบ synchronous จึงต้องดูโค้ดฝั่งผู้เรียกประกอบ เช่น PlaceOrder และ helper ใน system/online-boutique/src/checkoutservice/main.go:230–393

2. หากนักพัฒนาใหม่อ่านเฉพาะ ADR ที่กู้คืนมานี้ เขาอาจเข้าใจผิดในเรื่องใด

   **คำตอบ:** อาจเข้าใจว่าเหตุผลที่อนุมานเป็นเจตนาจริงของผู้ออกแบบ หรือคิดว่าทางเลือก HTTP/JSON และ message broker เคยถูกพิจารณาแล้ว ทั้งที่ ADR สร้างทางเลือกเหล่านี้เพื่อเปรียบเทียบและไม่มีหลักฐานประวัติการตัดสินใจ นอกจากนี้ อาจเหมารวมว่าทุกบริการใช้ gRPC แบบรอผล แม้พฤติกรรมที่ตรวจยืนยันได้จำกัดอยู่ในเส้นทาง checkout ของ system/online-boutique/src/checkoutservice/main.go ส่วน system/online-boutique/src/frontend/main.go:141–147 แสดงการสร้าง gRPC client connections แต่ไม่ได้แสดง implementation ของ handler ที่เรียกบริการ

3. AI กู้คืนอะไรจากโค้ดได้บ้าง และอะไรที่มีเพียงสถาปนิกผู้ออกแบบเดิมเท่านั้นที่รู้ ยกตัวอย่างหนึ่งตัวอย่างจาก ADR ของนิสิต

   **คำตอบ:** AI กู้คืนโครงสร้างและพฤติกรรมที่ปรากฏในโค้ดได้ เช่น PlaceOrder เรียกชำระเงินก่อนจัดส่ง และรอผลก่อนตอบกลับ (system/online-boutique/src/checkoutservice/main.go:252–279) แต่เหตุผลว่าเลือกวิธีนี้เพราะความเรียบง่าย ข้อจำกัดเวลา หรือข้อกำหนดอื่น ต้องอาศัยคำยืนยันจากผู้ออกแบบเดิมหรือบันทึกเพิ่มเติม ตัวอย่างคือ Claim 15 ใน ADR ที่ว่า “A อาจถูกเลือกเพื่อลดความซับซ้อนของ workflow” ซึ่งเป็น Inferred ไม่ใช่เหตุผลที่กู้คืนจากโค้ดได้โดยตรง
