**ภาษา:** ภาษาไทย | [English](01-recover-architecture.en.md)

# งานที่ 1: กู้คืนสถาปัตยกรรม (ประมาณ 15 นาที)

> **เชื่อมโยงกับการบรรยาย:** T7 การกู้คืนสถาปัตยกรรม, AICH2 (การตามรอย), AICH3 (บริบท) และข้อค้นพบจาก JSS ว่า 93% ของงานวิจัยไม่เคยตรวจสอบผลลัพธ์จาก AI
> **คำถาม:** AI กู้คืนสถาปัตยกรรมได้มากแค่ไหน และขึ้นอยู่กับหลักฐานที่ AI ได้อ่านมากเพียงใด

**ระหว่างทำงานนี้ ห้ามเปิด repository ต้นฉบับของ Online Boutique** เพราะ README ของต้นฉบับมีแผนภาพสถาปัตยกรรมที่นิสิตกำลังกู้คืนอยู่

ในแต่ละ run ให้เริ่ม **แชตหรือ session ใหม่** แล้วส่งข้อความด้านล่างให้ agent โดย agent จะทำตาม [`prompts/1-recover-architecture.md`](../prompts/1-recover-architecture.md) และเขียนผลลัพธ์ไว้ที่ `exercise/outputs/1<run>-architecture.md`

**Run A: เฉพาะ interface**
```
Follow prompts/1-recover-architecture.md
run: A
evidence: system/online-boutique/protos/demo.proto
```

**Run B: interface + ไฟล์ตั้งค่าการ deploy**
```
Follow prompts/1-recover-architecture.md
run: B
evidence: system/online-boutique/protos/demo.proto, system/online-boutique/kubernetes-manifests/
```

ระหว่างที่ agent ทำงาน ให้สังเกตว่า agent เปิดไฟล์ใดบ้าง หาก agent อ่านไฟล์นอกเหนือจากหลักฐานที่อนุญาต ให้บันทึกไว้ใน prompt log

## ผลลัพธ์

- Run A: [`outputs/1A-architecture.md`](outputs/1A-architecture.md)
- Run B: [`outputs/1B-architecture.md`](outputs/1B-architecture.md)

เปิดแต่ละไฟล์ด้วย **Markdown: Open Preview** (`Ctrl+Shift+V` / `⇧⌘V`) เพื่อดูแผนภาพ

## ตรวจสอบ Run B ด้วยตัวเอง (ส่วนที่สำคัญที่สุด)

> บันทึกการช่วยทำใบงาน: AI ช่วยตรวจไฟล์และร่างคำตอบด้านล่างจากผล Run A/B ที่มีอยู่แล้ว ไม่ได้รัน A/B ใหม่ในแชตนี้ นักศึกษาควรเปิดหลักฐานทบทวนก่อนส่ง; ✅ หมายถึงพบหลักฐานในไฟล์ ไม่ใช่ผลทดสอบระบบที่กำลังรัน

ตรวจสอบ dependency ในตารางของ Run B **อย่างน้อยห้ารายการ** เทียบกับไฟล์จริง โดยเปิดไฟล์ที่ AI อ้างถึงและหาหลักฐาน นิสิตสามารถตรวจซอร์สโค้ดของ service ได้ด้วย เช่น `src/checkoutservice/main.go`

| ผู้เรียก → ผู้ถูกเรียก | หลักฐานที่ AI อ้าง | ยืนยันแล้วหรือไม่ ✅ / ❌ / ❓ | สิ่งที่นิสิตพบ |
|---|---|---|---|
| checkoutservice → paymentservice | `system/online-boutique/kubernetes-manifests/checkoutservice.yaml:61–62` (`PAYMENT_SERVICE_ADDR`); `paymentservice.yaml:79–80` (port gRPC 50051) | ✅ | manifest กำหนดปลายทางตรงกับ Service; โค้ด `system/online-boutique/src/checkoutservice/main.go:370` เรียก `PaymentService.Charge` และ `PlaceOrder:252` เรียก `chargeCard` จริง |
| checkoutservice → emailservice | `system/online-boutique/kubernetes-manifests/checkoutservice.yaml:63–64` (`EMAIL_SERVICE_ADDR`); `emailservice.yaml:81–82` (port gRPC 5000) | ✅ | โค้ด `system/online-boutique/src/checkoutservice/main.go:380` เรียก `EmailService.SendOrderConfirmation`; `PlaceOrder:273–279` รอผลก่อนตอบกลับ แต่หากส่งผิดพลาดจะบันทึกคำเตือนและยังตอบสำเร็จ |
| checkoutservice → shippingservice | `system/online-boutique/kubernetes-manifests/checkoutservice.yaml:59–60` (`SHIPPING_SERVICE_ADDR`); `shippingservice.yaml:79–80` (port gRPC 50051) | ✅ | โค้ด `system/online-boutique/src/checkoutservice/main.go:314–315` เรียก `GetQuote` และ `:387` เรียก `ShipOrder`; จึงมีทั้งการขอราคาและการสั่งจัดส่ง ไม่ใช่เพียงการเชื่อมต่อที่ตั้งค่าไว้ |
| checkoutservice → cartservice | `system/online-boutique/kubernetes-manifests/checkoutservice.yaml:67–68` (`CART_SERVICE_ADDR`); `cartservice.yaml:80–81` (port gRPC 7070) | ✅ | โค้ด `system/online-boutique/src/checkoutservice/main.go:325` เรียก `GetCart` และ `:333` เรียก `EmptyCart`; `PlaceOrder:263` เรียกล้างตะกร้าหลังจัดส่งสำเร็จ |
| checkoutservice → currencyservice | `system/online-boutique/kubernetes-manifests/checkoutservice.yaml:65–66` (`CURRENCY_SERVICE_ADDR`); `currencyservice.yaml:80–81` (port gRPC 7000) | ✅ | โค้ด `system/online-boutique/src/checkoutservice/main.go:360` เรียก `CurrencyService.Convert`; ถูกใช้แปลงราคาจัดส่งที่ `:302` และราคาสินค้าที่ `:348` |

- **สิ่งที่ผิดหรือขาดหายไป** ใน Run B (dependency ที่ไม่มีจริง, dependency ที่ขาดไป, container ที่น่าสงสัย): ใน 5 dependency ที่ตรวจ ไม่พบเส้นที่ขัดกับ manifest หรือโค้ด แต่ยังสรุปไม่ได้ว่า dependency ทั้งระบบครบถ้วน แผนภาพ Run B ไม่แสดงลำดับการเรียกและพฤติกรรมเมื่อผิดพลาด ซึ่งผล Run B ระบุเป็นข้อจำกัดไว้แล้ว; โค้ด `system/online-boutique/src/checkoutservice/main.go:252–279` ช่วยยืนยันว่า checkout ชำระเงิน จัดส่ง ล้างตะกร้า และรอส่งอีเมลก่อนตอบ โดยการส่งอีเมลผิดพลาดไม่ทำให้คำสั่งซื้อล้มเหลว ส่วน Redis ใน `system/online-boutique/kubernetes-manifests/cartservice.yaml:141` ใช้ `emptyDir` จึงไม่ควรตีความรูป data store ว่ารับรองข้อมูลถาวรข้ามอายุ Pod; ประเด็นนี้ Run B ระบุไว้ใน Gaps แล้ว

## สะท้อนคิด (ข้อละ 2–3 ประโยค)

1. เปรียบเทียบ A กับ B: agent **ไม่สามารถ**รู้อะไรได้จากไฟล์ `.proto` เพียงไฟล์เดียว และเพราะเหตุใด

   จากผล Run A ไฟล์ `.proto` บอกชื่อบริการ RPC และโครงสร้างข้อมูล แต่ไม่บอกว่าใครเรียกใคร มี frontend หรือ Redis หรือแยก deploy อย่างไร จึงไม่ควรใช้ชนิดข้อมูลร่วมกันเป็นหลักฐานของ dependency ส่วน Run B มี manifest เพิ่ม จึงเห็น workload และปลายทางจากตัวแปรอย่าง `PAYMENT_SERVICE_ADDR` และ `REDIS_ADDR` แต่ยังต้องอ่านโค้ดเพื่อยืนยันจุดเรียก ลำดับ และการจัดการข้อผิดพลาด (`exercise/outputs/1A-architecture.md`: Dependencies table, Gaps; `exercise/outputs/1B-architecture.md`: Containers table, Dependencies table, Gaps)

2. ข้อผิดพลาด**ข้อใดข้อหนึ่ง**ที่จะทำให้สมาชิกใหม่ของทีมเข้าใจผิดมากที่สุด หากเผยแพร่แผนภาพนี้โดยไม่ตรวจสอบ

   ข้อผิดพลาดที่ควรระวังคือการตีความเส้น `checkoutservice → emailservice` ว่าอีเมลล้มเหลวแล้วคำสั่งซื้อต้องล้มเหลวด้วย เพราะโค้ด `system/online-boutique/src/checkoutservice/main.go:273–279` แสดงว่ารอผลส่งอีเมล แต่เมื่อผิดพลาดจะเพียงบันทึกคำเตือนและยังตอบสำเร็จ หากเข้าใจผิด สมาชิกใหม่อาจออกแบบการรับมือ outage หรือวิเคราะห์ความพร้อมใช้งานของ checkout ผิด โดย Run B ไม่ได้อ้างพฤติกรรมผิดนี้ แต่เส้น dependency เพียงอย่างเดียวให้ข้อมูลไม่พอ
