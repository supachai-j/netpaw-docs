# การใช้ skill

ไม่จำเป็นต้องรู้ชื่อ tool แค่อธิบายงานที่ต้องการ agent จะเลือก skill จากคำอธิบายของ skill แล้วเรียก tool เอง ผลลัพธ์ด้านล่างได้จากการเรียก tool จริง ส่วนความแม่นยำที่ model *จริง* เลือก skill ได้ถูกต้องวัดไว้ใน `docs/evals/TRIGGER-RESULTS.md` และยังไม่สมบูรณ์ (ดู [แนวคิด](concepts.html))

## คำนวณ subnet

*"ขอรายละเอียดของ 192.0.2.0/26 รวมถึง wildcard mask สำหรับ ACL"*

| ฟิลด์ | ค่า |
|---|---|
| network / broadcast | 192.0.2.0 / 192.0.2.63 |
| netmask / wildcard | 255.255.255.192 / 0.0.0.63 |
| usable hosts | 62 (192.0.2.1 - 192.0.2.62) |
| previous / next | 192.0.1.192/26 / 192.0.2.64/26 |
| classification | documentation (RFC 5737) |

prompt อื่น ๆ: *"ใน 2001:db8::/48 มี /64 กี่วง?"* (65,536) *"วาง VLSM สำหรับ 500, 100 และ 10 host ภายใน 10.20.0.0/16"* input ต้องเป็น network แบบ canonical ถ้าใส่ `192.0.2.5/26` จะถูกปฏิเสธ ไม่ใช่แก้ให้เงียบ ๆ

## รีวิว firewall rule

*"rule ใหม่นี้จะถูก shadow ไหม? rule เดิม: deny tcp 443 from 10.0.0.0/8 to 192.0.2.0/24 rule ใหม่: permit tcp 443 from 10.1.0.0/16 to 192.0.2.10/32"*

ผลลัพธ์: `overlap_type: shadowed` severity `critical` - rule deny ที่อยู่ก่อนและกว้างกว่าครอบ permit ใหม่ทั้งหมด permit นี้จึงไม่มีวัน match ลำดับที่แนะนำ: ใช้ `parse_policy` เพื่อ normalise config ของ vendor แล้วใช้ `analyze_firewall_rule_overlap` (หรือ `batch_analyze_overlap`) กับ rule ที่จะเพิ่ม จากนั้นให้คนอนุมัติการเปลี่ยนแปลง skill นี้ไม่แตะอุปกรณ์เลย parser ทำงานแบบ best effort ให้เทียบผลลัพธ์กับ config จริง และ sanitise config ก่อนส่งให้ model

## อ้างอิง RFC

*"RFC 4271 พูดถึงความปลอดภัยของ BGP session ว่าอย่างไร? ยก section มาด้วย"* agent จะค้นใน index ดึง section จาก rfc-editor.org แล้วตอบพร้อมข้อความที่ยกมาและหมายเลข section ข้อความที่ดึงมาถือเป็นข้อมูลที่ไม่น่าเชื่อถือ ถ้าในหน้า RFC มีคำสั่งแฝงอยู่ agent ควรเพิกเฉย

## ตรวจ lab ของ EVE-NG

วางข้อความจากไฟล์ `.unl` ที่ export ออกมา พร้อมความต้องการของคุณ (เช่น "core สองตัว distribution แบบ dual-homed ไม่มี host ต่อเข้ากับ core") tool จะรายงาน node ที่ไม่มีการเชื่อมต่อ link ที่ค้างอยู่ partition, adjacency ที่ห้ามมี single point of failure และจำนวน uplink tool เดา role จากชื่อ node (`CORE1`, `SW2`) ถ้าเดาผิดให้ส่ง `roles_json` เข้าไป tool ไม่ได้ตรวจ image, RAM, routing หรือความเร็ว interface

## ค้นข้อมูลพื้นฐาน

*"ขอประวัติของ OSPF พร้อมแหล่งอ้างอิง"* agent จะอ่าน Wikipedia ผ่าน `en.wikipedia.org/w/api.php` แล้วตอบเป็นสรุปพร้อม URL และหมายเหตุสัญญาอนุญาต CC BY-SA ถ้าต้องการรู้ว่ามาตรฐานกำหนดอะไรจริง ๆ ให้ใช้ rfc-lookup แทน

## ค้นข้อมูล BGP และ registry

*"เราได้รับ 193.0.0.0/24 จาก AS3333 มัน RPKI valid ไหม และใครเป็นเจ้าของ address space นี้?"* agent จะถาม validator ของ RIPE NCC เกี่ยวกับคู่นี้ แล้วถาม RDAP ของ registry ที่รับผิดชอบ กรณีนี้คาดว่าจะได้ `invalid` ด้วยเหตุผล `length` (ROA อนุญาตเฉพาะ /21) ส่วน `not_found` หมายถึงไม่มี ROA ซึ่งเป็นเรื่องปกติ ไม่ใช่ข้อค้นพบ address แบบ private, reserved และ documentation จะถูกปฏิเสธก่อนส่งข้อมูลใด ๆ ออกไป ดังนั้นให้ถามเฉพาะทรัพยากรสาธารณะ คำถามอื่นที่ตอบได้: mailbox สำหรับแจ้ง abuse ของ address, prefix ที่ AS หนึ่งประกาศ (มุมมองจาก RIPE RIS) และ network นั้น peer อยู่ที่ไหน (PeeringDB ข้อมูลที่เจ้าของรายงานเอง)

## Automation และการใช้แบบ headless

- Codex `exec` รัน tool แบบ read-only ได้ tool ที่ไม่มี `readOnlyHint` จะถูกปฏิเสธ
- thClaws `-p` และ OpenClaw `agent --local` รัน MCP tool **โดยไม่ถาม** แม้แต่ tool ที่เขียนข้อมูล นี่คือเหตุผลที่ netpaw ยังไม่มี tool ที่เขียนข้อมูล และเมื่อมีจะบังคับการอนุมัติภายใน server เอง
- คุณสั่ง server โดยตรงได้ด้วย `uv run --no-project --isolated --with mcp==1.30.0 python tests/smoke_mcp.py`
