ผู้จัดทำ: Codex; โมเดล: GPT-6

# Quality-attribute scenarios

ค่าตัวเลขด้านล่างมาจากข้อกำหนด ส่วนเหตุขัดข้องเป็นสถานการณ์ทดสอบที่เสนอ

| Source | Stimulus | Environment | Response | Response measure |
|---|---|---|---|---|
| ผู้ซื้อ | สั่งซื้อ 200 orders/s | ช่วง 11.11; MailFast มี p95 1.5 s, p99 8 s และผิดพลาด 0.5% | checkout บันทึก order/outbox และตอบกลับโดยไม่รอผู้รับงาน | PlaceOrder p95 ≤ 500 ms; อัตราส่งอีเมลรวม retry ≤ 50/s (docs/scenario.md:24–26) |
| relay/ผู้รับงาน | ส่ง OrderPlaced ซ้ำ หรือหยุดหลัง commit ledger ก่อน ack | มีคำสั่งซื้อสำเร็จและเกิดการส่งซ้ำ | loyaltyservice ประมวลผลซ้ำได้โดยไม่เพิ่มแต้มอีก | แต่ละคำสั่งซื้อได้รับแต้มครั้งเดียวภายใน 60 วินาที; จุดเริ่มจับเวลายังต้องกำหนด (docs/scenario.md:27; ADR: Decision, Validation) |
| บริการหลังสั่งซื้อ | email, loyalty หรือ analytics หยุดทำงาน | มีผู้ซื้อใช้งาน; ทดสอบแยกกรณี loyalty หยุดเกิน 60 วินาที | checkout ยังสำเร็จและเก็บงานไว้กู้คืน | ไม่มีคำสั่งซื้อล้มเหลวเพราะบริการเหล่านี้; แต้มที่เกิน 60 วินาทีต้องนับว่าผิดข้อกำหนด ไม่ใช่ผ่านเพราะกู้คืนได้ (docs/scenario.md:27–28) |

# Findings table

| Type | Finding | Why it matters |
|---|---|---|
| Non-risk | ข้อเสนอแยกคิวแต่ละงานและทำ ledger กับ deduplication แบบ atomic สอดคล้องกับการแยกความล้มเหลวและป้องกันแต้มซ้ำ (ADR: Decision) | เป็นกลไกที่เหมาะกับข้อกำหนดแต้มครั้งเดียวและไม่ให้บริการหลังสั่งซื้อขวาง checkout แต่ยังเป็นข้อเสนอที่ต้องพิสูจน์ (docs/scenario.md:27–28) |
| Risk | ยังไม่เลือก order store หรือกำหนดผลลัพธ์เมื่อเขียนไม่ได้; payment/shipping เกิดก่อน transaction ใหม่ (ADR: Decision, Open questions) | **[Inferred]** อาจชำระเงิน/ส่งสินค้าแล้วแต่ไม่มี order/event ถาวร หรือผู้ซื้อ retry แล้วเกิดผลซ้ำ; outbox ครอบคลุมเฉพาะข้อมูลใน transaction เดียว |
| Sensitivity point | ระยะพีกและโควตา MailFast กำหนดขนาด backlog: หากหนึ่งอีเมลต่อ order จะค้างอย่างน้อย 150 ฉบับ/s ก่อน retry (ADR: Consequences; docs/scenario.md:25–26) | **[Inferred]** พีกนาน T วินาทีสร้างงานค้างอย่างน้อย 150T ฉบับ; ยังระบุขนาดพื้นที่และเวลาระบายคิวไม่ได้หากไม่ทราบโหลดหลังพีกและเส้นตายอีเมล |
| Sensitivity point | เวลา relay, เวลารอคิว, retry และเวลาให้แต้มรวมกันต้องไม่เกิน 60 วินาที (ADR: Validation; docs/scenario.md:27) | **[Inferred]** แยกคิวอย่างเดียวไม่รับรอง deadline; หาก loyalty หยุดเกินเวลาที่เหลือ จะผ่านข้อกำหนดเดิมไม่ได้ แม้ checkout ยังพร้อมใช้ |
| Trade-off point | การเพิ่ม durable store/outbox ก่อนตอบสำเร็จช่วยรักษาเหตุการณ์ แต่เพิ่มงานในเส้นทาง checkout และภาระดูแล broker/store/ledger (ADR: Decision, Consequences) | **[Inferred]** แลกความทนทานของข้อมูลกับ latency, dependency และต้นทุนทีม 4 คน; ต้องวัดทั้งเส้นทางที่ 200 orders/s (docs/scenario.md:24–25, 29–30) |

# Missing or weak

- ทางเลือกสามข้อแตกต่างกันจริง แต่เหตุผลที่เลือก RabbitMQ เหนือตัวประสานงานเบื้องหลังยังเป็นเชิงคุณภาพ ควรเทียบภาระพัฒนา/ดูแลและการเพิ่มงานใหม่ภายใต้โหลดเดียวกัน (ADR: Considered options; docs/scenario.md:14, 29–30) ไม่จำเป็นต้องเพิ่มผลิตภัณฑ์เพียงเพื่อให้มีตัวเลือกมากขึ้น
- ADR ระบุปัญหาไว้แล้ว แต่ยังไม่ปิดเรื่อง reconciliation, idempotency ของการสั่งซื้อ, พื้นที่ outbox เต็ม และ SLA แต้มช่วง outage; เป็นเงื่อนไขก่อนยอมรับ ไม่ใช่เพียงรายละเอียดหลังเลือกเทคโนโลยี (ADR: Decision, Open questions)
- การเพิ่มผู้รับงานโดยไม่แก้ checkout ขึ้นกับว่า event มีข้อมูลเพียงพอ; SMS/fraud/recommendations อาจต้องเปลี่ยน schema **[Inferred]** จาก docs/scenario.md:14 และ ADR: Decision ข้อสัญญาเหตุการณ์
- โค้ดและ ADR-0001 ที่ ADR อ้างถึงอยู่นอกหลักฐานที่อนุญาต จึงยังตรวจยืนยันข้ออ้างจากแหล่งเหล่านั้นไม่ได้; รายละเอียดการทำงานเกินกว่าคำอธิบายใน docs/scenario.md:18 เป็น **Not evidenced in the provided files** สำหรับรีวิวนี้

# Verdict

**accept with changes** — ยอมรับทิศทาง โดยต้องปิดสามประเด็นก่อนอนุมัติใช้งาน:

1. เลือก store และกำหนดขอบเขต transaction, order/request identity, reconciliation รวมถึงผลตอบกลับเมื่อเขียนไม่ได้หรือพื้นที่เต็ม พร้อมเกณฑ์ทดสอบเหตุขัดข้อง
2. ยืนยันจุดเริ่มและขอบเขต SLA แต้ม 60 วินาที โดยให้เจ้าของข้อกำหนดตัดสินกรณี outage; กำหนดงบเวลา relay/queue/retry และพิสูจน์ว่าให้แต้มครั้งเดียวที่ 200 orders/s
3. กำหนดระยะพีก โหลดหลังพีก และเส้นตายอีเมล เพื่อนำไปคำนวณ backlog/retention/โควตา พร้อมระบุเจ้าของการดูแลและวัด checkout p95 หลังเพิ่ม persistence
