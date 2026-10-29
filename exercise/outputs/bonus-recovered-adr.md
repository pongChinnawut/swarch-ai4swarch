ผู้จัดทำ: Codex; รุ่นโมเดล: ไม่ทราบรหัสรุ่นที่แน่นอน

# ADR-BONUS: ใช้ gRPC แบบรอผลและรวมสัญญาใน demo.proto

- **Status:** Reconstructed
- **Date:** 2026-10-29 (วันที่กู้คืน ไม่ใช่วันที่ตัดสินใจเดิม)
- **Deciders:** ไม่ปรากฏในหลักฐาน

## Context and problem

คำถามที่กู้คืน: “Why do all services communicate synchronously over gRPC, using one shared demo.proto?”

ขอบเขต: ไฟล์ที่ให้มายังยืนยันคำว่า “all services” ไม่ได้ รวมถึงไม่ยืนยันกระบวนการสร้างโค้ดของทุกบริการจากไฟล์เดียว เหตุผลดั้งเดิม ผู้ตัดสินใจ วันที่ตัดสินใจ และทางเลือกที่เคยพิจารณาจริง: ไม่ปรากฏหลักฐานในไฟล์ที่ให้มา (Not evidenced in the provided files) ทางเลือกด้านล่างสร้างขึ้นเพื่อเปรียบเทียบ ไม่ใช่ประวัติการตัดสินใจ

เลข [1]–[18] อ้างถึงหลักฐานและประเภทของข้อกล่าวอ้างใน **Claim check**

- **[1 — Evidenced]** demo.proto ใช้ proto3 และรวมสัญญา unary RPC ของเก้าบริการไว้ในไฟล์เดียว ไม่มีคำประกาศ stream แต่รูปแบบสัญญาเพียงอย่างเดียวไม่พิสูจน์ว่าผู้เรียกทุกตัวรอผลแบบ synchronous
- **[2 — Evidenced]** หลายสัญญาใช้ Money ร่วมกัน เช่น Product.price_usd, ChargeRequest.amount และ CurrencyConversionRequest.from
- **[3 — Evidenced]** checkoutservice เปิด gRPC server และสร้าง gRPC client connections ไปยัง shipping, product catalog, cart, currency, email และ payment
- **[4 — Evidenced]** PlaceOrder รอขั้นเตรียมสินค้า ชำระเงิน จัดส่ง ล้างตะกร้า และส่งคำยืนยันก่อนตอบกลับ โดย helper เรียก RPC แล้วใช้ผลลัพธ์หรือ error ต่อทันที
- **[5 — Evidenced]** frontend/main.go สร้าง gRPC client connections ไปเจ็ดบริการธุรกิจ และรับคำขอเว็บผ่าน HTTP; ไฟล์นี้ไม่ได้แสดง implementation ของ handler ที่เรียกบริการ

## Decision drivers

ลำดับต่อไปนี้เป็นการอนุมาน ไม่ใช่ข้อกำหนดที่พบในหลักฐาน

1. **[6 — Inferred]** ความเข้าใจง่ายและ modifiability ของ workflow checkout อาจเป็นแรงจูงใจให้ใช้ลำดับเรียกแล้วรอผล
2. **[7 — Inferred]** ความสอดคล้องของสัญญาข้อมูลอาจเป็นแรงจูงใจให้รวม schema
3. **[8 — Inferred]** การได้ผลชำระเงินและหมายเลขติดตามก่อนตอบคำสั่งซื้ออาจเป็นแรงจูงใจด้าน functional suitability

## Considered options

1. **A:** gRPC แบบรอผลสำหรับเส้นทาง checkout และสัญญารวมใน demo.proto
2. **B:** HTTP/JSON แบบรอผล และสัญญาแยกตามบริการ
3. **C:** workflow สั่งซื้อแบบ asynchronous ผ่าน message broker และสัญญา event แยก

### Pros and cons of the options

ข้อดีข้อเสียเป็นความรู้ทั่วไปสำหรับเปรียบเทียบ ไม่ใช่ผลทดสอบของ Online Boutique

| Option | Pros | Cons |
|---|---|---|
| A | **[9 — Assumption]** สัญญาที่มีชนิดข้อมูลและเครื่องมือสร้างโค้ดช่วยลดงานเขียนโค้ดเชื่อมต่อซ้ำ (modifiability) | **[10 — Assumption]** การรอผลผูก latency กับปลายทาง และการเปลี่ยนสัญญาร่วมที่ไม่เข้ากันอาจกระทบหลายผู้ใช้ (performance, modifiability) |
| B | **[11 — Assumption]** HTTP/JSON ตรวจด้วยเครื่องมือเว็บทั่วไปได้สะดวก และสัญญาแยกช่วยกำหนดขอบเขตการเปลี่ยนแปลง (operability, modifiability) | **[12 — Assumption]** ยังคงรอปลายทาง และต้องดูแลความสอดคล้องของข้อมูลข้ามสัญญาแยก (performance, modifiability) |
| C | **[13 — Assumption]** หากมีคิว durable ผู้ผลิตสามารถฝากงานระหว่างผู้รับหยุดชั่วคราวได้ (availability) | **[14 — Assumption]** ต้องดูแล broker และออกแบบสถานะ งานซ้ำ และการชดเชยความสำเร็จบางส่วน (cost, reliability, complexity) |

## Decision

**[1–4 — Evidenced]** กู้คืนรูปแบบที่สังเกตได้เป็น A เฉพาะเส้นทาง checkout ที่ตรวจ: เรียก gRPC แบบรอผล และใช้ข้อความที่มีสัญญาประกาศใน demo.proto การนำเข้า genproto ไม่ได้ยืนยันขั้นตอนสร้างโค้ดจากไฟล์นี้ของทุกบริการ

**[15 — Inferred]** เหตุผลที่เป็นไปได้คือ A สอดคล้องกับการใช้ผลลัพธ์ระหว่างขั้นตอนและชนิดข้อมูลร่วม จึงอาจลดความซับซ้อนในการเขียน workflow ไม่ปรากฏหลักฐานว่าผู้ออกแบบเลือกด้วยเหตุผลนี้ หรือเคยปฏิเสธ B และ C

## Consequences

- **Positive — [7,15 — Inferred]:** สัญญาร่วมและลำดับเรียกที่ชัดเจนอาจช่วยติดตามข้อมูลและแก้ไข workflow ได้ง่ายขึ้น (quality attribute: modifiability) เป็นประโยชน์คาดหมาย ไม่ใช่ผลวัด
- **Negative — [16 — Inferred]:** latency ของการเรียกต่อเนื่อง รวมถึงอ่านสินค้าและแปลงราคาทีละรายการ อาจสะสมในเวลาตอบ checkout (quality attribute: performance) ไม่มีตัวเลข latency ในหลักฐาน
- **Risks and mitigations — [17 — Evidenced]:** เมื่อ chargeCard คืน error คำสั่งซื้อจะตอบ error; เมื่อ shipOrder คืน error จะตอบ Unavailable แม้ขั้นชำระเงินก่อนหน้าจะคืนสำเร็จแล้ว ส่วน error จากล้างตะกร้าถูกละทิ้ง และ error จากส่งคำยืนยันถูกบันทึกแล้วตอบสำเร็จต่อ (quality attribute: reliability, availability) ข้อนี้แสดง control flow ไม่ได้ยืนยันผลการตัดเงินจริงหรือกลไกชดเชยภายในปลายทาง
- **Risks and mitigations — [18 — Assumption]:** เสนอให้ประเมิน deadline และการส่งต่อ cancellation ทดสอบ idempotency ก่อนเพิ่ม retry ในงานชำระเงิน ออกแบบการชดเชยความสำเร็จบางส่วน และตรวจ backward compatibility ก่อนเปลี่ยน schema ร่วม (quality attribute: performance, reliability, modifiability, interoperability) เป็นข้อเสนอ ไม่ได้ยืนยันว่ามีมาตรการแล้ว

## Validation

**[18 — Assumption; แผนที่เสนอ ยังไม่ได้ดำเนินการ]**

- ให้ผู้ออกแบบเดิมยืนยัน driver เหตุผล ทางเลือก และขอบเขตคำว่า “ทุกบริการ”
- ทดสอบ checkout เมื่อปลายทางช้าหรือล้มเหลว วัด latency และอัตราสำเร็จเทียบเกณฑ์ที่ผู้รับผิดชอบกำหนด (quality attribute: performance, availability)
- ทดสอบกรณีชำระเงินสำเร็จแต่จัดส่งล้มเหลว รวมถึงคำขอซ้ำ (quality attribute: reliability)
- ตรวจความเข้ากันได้ของสัญญาและกระบวนการสร้างโค้ด (quality attribute: interoperability, modifiability)

ยังไม่ได้รันทดสอบ และไม่ปรากฏเป้าหมาย latency, availability หรือผล benchmark ในไฟล์ที่ให้มา

## Claim check

เลขเดียวกันที่ปรากฏซ้ำหมายถึงข้อกล่าวอ้างเดียวกัน แต่ละแถวรวมข้อความที่มีประเภทและฐานหลักฐานเดียวกัน

| # | Claim | Type | Evidence |
|---|---|---|---|
| 1 | proto3 รวมสัญญา unary RPC ของเก้าบริการ | Evidenced | system/online-boutique/protos/demo.proto:15–244 — syntax; CartService, RecommendationService, ProductCatalogService, ShippingService, CurrencyService, PaymentService, EmailService, CheckoutService, AdService; ไม่พิสูจน์รูปแบบรอผลของผู้เรียกทุกตัว |
| 2 | หลายสัญญาใช้ Money ร่วมกัน | Evidenced | system/online-boutique/protos/demo.proto:82,146–161,169,189 — Product, Money, CurrencyConversionRequest, ChargeRequest |
| 3 | checkout เปิด gRPC server และสร้าง client connections ไปหกบริการ | Evidenced | system/online-boutique/src/checkoutservice/main.go:110–146,210–219 — main, mustConnGRPC, RegisterCheckoutServiceServer |
| 4 | checkout รอผลขั้นตอนและ RPC ก่อนตอบ | Evidenced | system/online-boutique/src/checkoutservice/main.go:230–279,288–393 — PlaceOrder, prepareOrderItemsAndShippingQuoteFromCart, getUserCart, quoteShipping, prepOrderItems, convertCurrency, chargeCard, emptyUserCart, sendOrderConfirmation, shipOrder; main.go:34 นำเข้า genproto แต่ไม่แสดงขั้นตอนสร้างโค้ด |
| 5 | frontend สร้างเจ็ด gRPC client connections และรับเว็บผ่าน HTTP | Evidenced | system/online-boutique/src/frontend/main.go:132–171 — main, mustConnGRPC, http.ListenAndServe; ไม่มี implementation ของ handler ในไฟล์นี้ |
| 6 | ความเข้าใจง่ายของ workflow อาจเป็น driver | Inferred | อนุมานจาก system/online-boutique/src/checkoutservice/main.go:230–279 — PlaceOrder; ไม่บันทึกเจตนาผู้ออกแบบ |
| 7 | ความสอดคล้องของชนิดข้อมูลอาจเป็น driver และช่วยการแก้ไข | Inferred | การใช้ Money ใน system/online-boutique/protos/demo.proto:82,146–161,169,189; ไม่มีหลักฐานแรงจูงใจหรือผลด้าน modifiability |
| 8 | ต้องการผลชำระเงินและติดตามพัสดุก่อนตอบอาจเป็น driver | Inferred | system/online-boutique/src/checkoutservice/main.go:252–279 — PlaceOrder; system/online-boutique/protos/demo.proto:208–214 — OrderResult; ไม่ใช่ข้อกำหนดที่ระบุไว้ |
| 9 | เครื่องมือสร้างโค้ดจากสัญญาอาจลดงานเขียนโค้ดเชื่อมต่อ | Assumption | ความรู้ทั่วไป; ไม่ปรากฏหลักฐานผลประหยัดงานหรือขั้นตอนสร้างโค้ดในไฟล์ที่ให้มา |
| 10 | การรอผลผูก latency กับปลายทาง; สัญญาร่วมที่เปลี่ยนแบบไม่เข้ากันเสี่ยงกระทบผู้ใช้ | Assumption | ความรู้ทั่วไปเพื่อประเมิน A; ไม่ปรากฏหลักฐานผลกระทบจริงหรือการเปรียบเทียบในไฟล์ที่ให้มา |
| 11 | HTTP/JSON ตรวจด้วยเครื่องมือเว็บได้สะดวก; สัญญาแยกช่วยกำหนดขอบเขตการเปลี่ยน | Assumption | ความรู้ทั่วไปเพื่อประเมิน B; ไม่ปรากฏหลักฐานว่าเคยประเมิน B |
| 12 | HTTP แบบรอผลยังรอปลายทาง; สัญญาแยกต้องดูแลความสอดคล้องข้ามสัญญา | Assumption | ความรู้ทั่วไปเพื่อประเมิน B; ไม่ใช่ผลทดลองของระบบ |
| 13 | คิว durable ช่วยฝากงานระหว่างผู้รับหยุดชั่วคราว | Assumption | ความรู้ทั่วไปแบบมีเงื่อนไขเพื่อประเมิน C; ไม่ปรากฏหลักฐานว่าระบบใช้ broker |
| 14 | workflow ผ่าน broker เพิ่มภาระดูแลและออกแบบสถานะ งานซ้ำ และการชดเชย | Assumption | ความรู้ทั่วไปเพื่อประเมิน C; ไม่ปรากฏหลักฐานว่าเคยประเมิน C |
| 15 | A อาจถูกเลือกเพื่อลดความซับซ้อนของ workflow | Inferred | อนุมานจากข้อ 2 และ 4; ไม่มีบันทึกเหตุผลดั้งเดิมหรือผลเปรียบเทียบทางเลือก |
| 16 | การเรียกต่อเนื่องและวนทีละสินค้าอาจสะสม latency | Inferred | system/online-boutique/src/checkoutservice/main.go:238–279,339–356 — PlaceOrder, prepOrderItems; ไม่มีผลวัด |
| 17 | error จากชำระเงินหรือจัดส่งทำให้ตอบ error; ล้างตะกร้าและอีเมลจัดการต่างกัน | Evidenced | system/online-boutique/src/checkoutservice/main.go:252–279 — PlaceOrder; แสดงเพียง control flow ฝั่ง checkout |
| 18 | มาตรการและแผนตรวจสอบที่เสนออาจช่วยประเมินและลดความเสี่ยง | Assumption | ข้อเสนอจากหลักทั่วไปด้าน performance, availability, reliability และ modifiability; ไม่ปรากฏผลทดสอบ เกณฑ์ยอมรับ หรือการยืนยันว่ามีมาตรการแล้ว |

จำนวนข้อกล่าวอ้าง: **18** — **Evidenced 6**, **Inferred 5**, **Assumption 7** เป็นการจัดประเภทของ agent สำหรับให้นิสิตตรวจสอบต่อ
