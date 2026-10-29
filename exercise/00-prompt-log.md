**ภาษา:** ภาษาไทย | [English](00-prompt-log.en.md)

# บันทึกการใช้ prompt

**ชื่อ / รหัสนิสิต:** นาย ชินนวุฒิ แก้วฉิม (6870063021)
**agent ที่ใช้:** Codex 
**โมเดล:** GPT-6.1 Sol
**แพ็กเกจที่ใช้:** GPT Package Pro

บันทึกทุกคำขอที่ส่งให้ agent ช่อง "ไฟล์ที่ agent อ่าน" คือไฟล์ที่ agent รายงาน หรือที่นิสิตเห็นว่า agent เปิด

| # | งาน / run | agent + โมเดล | ข้อความที่ส่ง (ย่อ) | ไฟล์ที่ agent อ่าน | อยู่ในขอบเขตหลักฐานที่อนุญาตหรือไม่ | มีประโยชน์ (1–5) | หมายเหตุ |
|---|---|---|---|---|---|---|---|
| 1 | 1-A | Codex / GPT-6.1 Sol | `prompts/1-…` run A | AGENTS.md, prompts/1-recover-architecture.md, system/online-boutique/protos/demo.proto | OK | 4 | |
| 2 | 1-B | Codex / GPT-6.1 Sol | `prompts/1-…` run B | AGENTS.md, prompts/1-recover-architecture.md, system/online-boutique/protos/demo.proto, ภายใต้ system/online-boutique/kubernetes-manifests/:- README.md, adservice.yaml, cartservice.yaml, checkoutservice.yaml, currencyservice.yaml, emailservice.yaml, frontend.yaml, kustomization.yaml, loadgenerator.yaml, paymentservice.yaml, productcatalogservice.yaml, recommendationservice.yaml, shippingservice.yaml | OK | 5 | |
| 3 | 2-a | Codex / GPT-6.1 Sol | `prompts/2-…` run a | prompts/2-write-adr.md, docs/adr/template.md, exercise/outputs/2a-adr.md | OK | 5| |
| 4 | 2-b | Codex / GPT-6.1 Sol | `prompts/2-…` run b | prompts/2-write-adr.md, docs/adr/template.md, docs/scenario.md, docs/adr/0001-use-redis-for-cart-storage.md, system/online-boutique/src/checkoutservice/main.go | OK | 5 | |
| 5 | 3 | Codex / GPT-6.1 Sol | `prompts/3-…` | AGENTS.md, prompts/3-review-adr.md, docs/adr/0002-use-rabbit.md, docs/scenario.md| OK | 4 | |
| 6 | | | | | | | |
| 7 | | | | | | | |
