> หน้านี้เป็นคำแปลภาษาไทยของคู่มือภาษาอังกฤษ ([English](../index.html)) ถ้าเนื้อหาไม่ตรงกัน ให้ถือฉบับภาษาอังกฤษเป็นหลัก

# เอกสาร netpaw

netpaw รวม skill ด้านงาน network engineering ไว้ให้ AI agent ใช้ **skill** คือไฟล์คำสั่งสั้น ๆ (`SKILL.md` ตามรูปแบบเปิด [Agent Skills](https://agentskills.io)) ที่บอก agent ว่าควรใช้ **tool** ชุดหนึ่งเมื่อไรและใช้อย่างไร ส่วน tool คือ [MCP](https://modelcontextprotocol.io) server ขนาดเล็กที่รันในเครื่องผ่าน stdio ซอร์สชุดเดียวติดตั้งได้ทั้งใน Claude Code, Codex, Hermes Agent, OpenClaw และ thClaws

## เริ่มจากตรงไหน

1. [เริ่มต้นใช้งาน](getting-started.html) - ติดตั้งให้ agent ของคุณภายในห้านาที
2. [การใช้ skill](using.html) - ตัวอย่าง prompt และผลลัพธ์จริง
3. [รายการ skill](skills.html) - tool ทุกตัว กฎของ input การเข้าถึงเครือข่าย และข้อจำกัด
4. [แนวคิด](concepts.html) - ส่วนต่าง ๆ ทำงานร่วมกันอย่างไร ระดับความพร้อม และทำไมจึงเป็น read-only
5. [Agent ที่รองรับ](agents.html) - ทดสอบอะไรกับ agent ตัวไหนแล้ว และอะไรที่ยังไม่ได้ทดสอบ
6. [ความปลอดภัย](safety.html) - threat model, control และความเสี่ยงที่ยังเหลืออยู่
7. [การพัฒนา](develop.html) - เพิ่ม skill หรือ MCP server
8. [การแก้ปัญหา](troubleshoot.html)

หน้าภาษาไทย: [ภาษาไทย](index.html) (อาจอัปเดตช้ากว่าหน้าภาษาอังกฤษ)

## สถานะโดยสรุป

| หัวข้อ | สถานะ |
|---|---|
| Skill | 5 ตัว ทั้งหมดเป็น `experimental` และ read-only ทั้งหมด |
| Agent ที่ทดสอบแล้ว | Claude Code, Hermes Agent (runtime จริง); Codex, OpenClaw, thClaws (mock model) |
| ผลกับ model จริง | มีเฉพาะ Claude Code ส่วน agent อื่นยังไม่มีหลักฐานว่าเลือกใช้ skill ได้เมื่อใช้ LLM จริง |
| การแก้ไขอุปกรณ์ | ไม่มี ไม่มี skill ใดเขียนค่าลงอุปกรณ์ |
| แพลตฟอร์ม | Ubuntu 24.04 x86_64 เท่านั้น |
| การเผยแพร่ | GitHub เท่านั้น ไม่มีบน PyPI หรือ npm |

ให้อ่านสถานะนี้เป็นข้อจำกัด ไม่ใช่ว่า "รองรับเต็มรูปแบบ" หลักฐานอยู่ใน [docs/CONFORMANCE.md](https://github.com/supachai-j/netpaw/blob/main/docs/CONFORMANCE.md) และ [docs/READINESS.md](https://github.com/supachai-j/netpaw/blob/main/docs/READINESS.md)

## การจัดการข้อมูล

ตัวอย่างทั้งหมดใช้ช่วง address สำหรับเอกสาร (192.0.2.0/24, 198.51.100.0/24, 203.0.113.0/24, 2001:db8::/32) อย่าใส่ชื่อลูกค้า hostname, IP หรือ config อุปกรณ์จริงลงใน prompt, issue หรือ commit ข้อมูลใดก็ตามที่ให้ agent อาจถูกส่งออกจากเครื่องไปยังผู้ให้บริการ model ของ agent นั้น ควร sanitise config ของ firewall ก่อนเสมอ
