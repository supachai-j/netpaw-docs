# ความปลอดภัย

อ้างอิง: OWASP Top 10 for LLM applications, OWASP MCP security guidance, NIST SSDF และ MCP specification บันทึกการรีวิวฉบับเต็มพร้อมป้ายกำกับหลักฐาน (VERIFIED / INTERPRETIVE / NOT CHECKED) อยู่ที่ `docs/SECURITY-REVIEW.md` การแจ้งช่องโหว่ให้ทำตาม `SECURITY.md`

## Threat model โดยย่อ

agent อ่านข้อความที่เชื่อถือไม่ได้ (RFC, Wikipedia, output จากอุปกรณ์, config ของ firewall, ไฟล์ `.unl`) และรัน tool ด้วยสิทธิ์ของ user ของคุณ ความเสี่ยงคือ prompt injection ผ่านข้อความที่ดึงมา, tool ทำมากกว่าที่บอกไว้, นิยามของ tool เปลี่ยนหลังจากที่คุณรีวิวไปแล้ว, dependency ถูกวางยา, และ secret หรือข้อมูลลูกค้ารั่วไปถึงโมเดลหรือเข้าไปอยู่ใน repo

## มาตรการควบคุม

1. **อ่านอย่างเดียวเป็นค่าเริ่มต้น** skill ประกาศ `risk: read|write` และ server ประกาศ `capability: read|write` ส่วน lint จะปฏิเสธ skill แบบ read ที่ใช้ server แบบ write ทุก tool ในปัจจุบันตั้ง `readOnlyHint` และไม่มีตัวไหนต้องใช้ credential
2. **การอนุมัติอยู่ที่ server** จากการวัดจริง: Codex `exec` บล็อก write tool ที่ยังไม่ได้อนุมัติ แต่ thClaws `-p` และ OpenClaw `--local` รันไปเลยโดยไม่ถาม ดังนั้น write tool ในอนาคตต้องบังคับเองทั้ง change request, baseline, การอนุมัติที่ผูกกับอุปกรณ์และ payload, การ verify และ audit ข้อความใน skill และ prompt ขออนุมัติของ agent ไม่นับเป็นมาตรการควบคุม
3. **ประกาศ egress** แต่ละ server ระบุ host ที่อนุญาตให้ติดต่อ rfc-lookup และ wikipedia ใช้ host เดียวต่อ server ผ่าน HTTPS, ปฏิเสธ redirect, บังคับ deadline รวม 15 s และจำกัดขนาด bgp-intel ใช้ public registry host เก้าตัว โดยมี path allow-list ตายตัวต่อ host, ปฏิเสธ resource แบบ private และ reserved ก่อนส่งอะไรออกไป และตัดข้อมูลติดต่อส่วนบุคคลออกจากคำตอบของ registry ถ้ามีการ import network module โดยไม่ได้ประกาศไว้ lint จะ fail
4. **ตรวจ input เข้มงวด** ทุก tool ปฏิเสธ field ที่ไม่รู้จัก, มีการ validate identifier (เลข RFC, ชื่อหน้า, CIDR), ไม่มีการอ่านไฟล์ตาม path (eve-lab รับเป็นข้อความและปฏิเสธ `DOCTYPE`/`ENTITY` ก่อน parse ส่วน fwrule proxy ทำแบบเดียวกันกับ XML declaration และจำกัดขนาด ruleset ที่ 4 MiB)
5. **output ถือว่าเชื่อถือไม่ได้** ข้อความจากอินเทอร์เน็ตถูกติดป้าย `untrusted_external_text` และตัดอักขระควบคุม, bidi, zero-width, variation-selector และ Unicode tag ออก วิธีนี้ลดความเสี่ยงได้ แต่ไม่ได้ทำให้ข้อความน่าเชื่อถือ การทดสอบ injection กับโมเดลจริงด้วยหลาย payload (instruction override, system tag ปลอม, ล่อให้เรียก tool, เนื้อหาที่ถูกแก้) ไม่พบว่าโมเดลทำตาม แต่ทดสอบกับโมเดลตระกูลเดียวและจำนวนตัวอย่างน้อย เนื้อหาที่ถูกแก้จะถูกกันได้ด้วย host ที่ pin ไว้และ TLS เท่านั้น ไม่ใช่ด้วยโมเดล
6. **pin และ lock** ใช้ `uv run --no-project --isolated` (project ใน working directory จะรัน build backend ของตัวเองไม่ได้), `mcp` เวอร์ชันตายตัว, requirement ของ fwrule ล็อกด้วย hash (ทุก server เริ่มทำงานหลัง shared proxy ซึ่งตอบจาก `tools/list` ที่ cache ไว้และ pin hash ไว้ใน config ของ agent ดังนั้นการแก้ไฟล์ cache อย่างเดียวจะถูกปฏิเสธ และก่อน forward ทุก call จะเทียบ tool list จริงกับ cache), GitHub Actions pin ด้วย commit, `tools/list` ทั้งหมดของทุก server ล็อกด้วย hash และตรวจใน CI ดังนั้นถ้านิยามของ tool เปลี่ยน build จะ fail
7. **job ด้านความปลอดภัยใน CI** gitleaks, bandit, semgrep (rule ของ project), pip-audit, zizmor, validation ด้วย `skills-ref` ทางการ, SBOM และ release มี build provenance
8. **สแกน skill** รัน Cisco Skill Scanner และ NVIDIA SkillSpector แบบ offline กับทุก skill พร้อม control skill ที่จงใจทำให้เป็นอันตราย เพื่อพิสูจน์ว่า scanner ตรวจจับได้จริง scanner ทั้งสองอ่านเฉพาะโฟลเดอร์ skill ไม่ได้อ่านโค้ดใน `mcp/`
9. **ไม่มี secret หรือข้อมูลลูกค้าใน repo** lint ตรวจหา pattern ของ credential อุปกรณ์เครือข่าย ตัวอย่างใช้เฉพาะช่วง address สำหรับเอกสาร

## สิ่งที่พบและแก้ไขแล้ว

ตัวอย่างที่คัดมา: `uv` รัน build backend ของ project จาก working directory (แก้ด้วย `--no-project --isolated` และบังคับด้วย lint); CGNAT ถูกจัดเป็น global; symlink และไฟล์หลงถูกคัดลอกเข้า build; dependency ไม่ได้ pin; ช่องทางเลี่ยงการตัด DOCTYPE ใน parser PAN-OS ของ fwrule ต้นทาง (บรรเทาใน proxy ของเรา **แต่ยังไม่ได้แก้ที่ต้นทาง**); deadline ที่คุม server ที่ตอบช้าไม่ได้; regex ของ header ที่ใช้เวลาแบบ super-linear; อักขระซ่อนที่หลุดรอดใน search path รายละเอียดและหลักฐานการ reproduce อยู่ที่ `docs/SECURITY-REVIEW.md` และ `docs/security-evidence/`

## ความเสี่ยงที่ยังเหลือ

- parser ของ fwrule ต้นทาง (นอกเหนือจากตัวกันใน proxy) ยังไม่ได้ทำ fuzzing และ repo ต้นทางไม่มีไฟล์ LICENSE
- การเริ่มครั้งแรกจะดาวน์โหลด package จาก PyPI (fwrule บังคับตรวจ hash ส่วน server เล็กที่ใช้ stdlib ต้องการแค่ package `mcp`)
- มีเพียง Claude Code ที่รันกับ tool ด้วยโมเดลจริง agent อื่นรันด้วย mock model
- ยังไม่มี write skill ดังนั้น approval gate ยังไม่เคยถูกทดสอบกับอุปกรณ์จริง
- ยังไม่ได้ตรวจสถานะเครื่องหมายการค้าของชื่อ `netpaw` และมี tool บน Windows ที่ชื่อคล้ายกันอยู่

## เมื่อใช้งาน

- ใช้ข้อมูล lab หรือ config ที่ sanitise แล้ว อะไรก็ตามที่ส่งให้ agent อาจไปถึงผู้ให้บริการโมเดลของ agent นั้น อย่าวางชื่อลูกค้า ข้อมูลติดต่อส่วนบุคคล credential หรือ address จริง
- ถือว่าผลสแกนหรือรีวิวที่ผ่านเป็นหลักฐาน ไม่ใช่ข้อพิสูจน์ ทุกอย่างที่เปลี่ยนแปลงเครือข่ายต้องมีคนอนุมัติ
- เปิด agent เฉพาะใน directory ที่คุณเชื่อถือ
