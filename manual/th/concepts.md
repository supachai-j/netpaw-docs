# แนวคิดหลัก

## Skill, tool, server

- **skill** (`skills/<name>/SKILL.md`) คือไฟล์ Markdown ที่มี YAML header สั้น ๆ ช่อง `description` ใน header คือสิ่งที่ agent อ่านเพื่อตัดสินใจว่าจะใช้หรือไม่ ส่วนเนื้อหาบอกวิธีใช้ รูปแบบเป็นไปตาม spec เปิดของ Agent Skills และตรวจใน CI ด้วย validator ทางการ `skills-ref`
- **tool** คือฟังก์ชันที่ agent เรียกได้ (เช่น `same_subnet`)
- **MCP server** (`mcp/<name>/`) คือ process ในเครื่องที่ให้บริการ tool ผ่าน stdio โดยมี `mcp/servers.json` เป็น registry เดียว และ `scripts/build.py` แปลงเป็นรูปแบบ config ของแต่ละ agent

skill ที่ไม่มี server เป็นแค่คำแนะนำ ส่วน server ที่ไม่มี skill ใช้งานได้ แต่ agent อาจไม่รู้ว่าควรใช้เมื่อไร netpaw จึงส่งมาเป็นคู่เสมอ

## แหล่งเดียว ใช้ได้หลาย agent

`skills/` และ `mcp/servers.json` ไม่ผูกกับ agent ตัวใด `bin/netpaw install` (หรือ `scripts/build.py`) คัดลอก skill ไปไว้ใน skill directory ของแต่ละ agent และเขียนหรือ merge MCP config ตามรูปแบบของ agent นั้น (TOML, YAML, JSON, CLI) ชื่อ tool ใน `SKILL.md` เขียนแบบไม่มี prefix แต่ละ agent จะเติม prefix ของตัวเอง

## ทำไมเริ่มจาก read-only

model ถูกชักนำได้ด้วยข้อความที่มันอ่าน (หน้า RFC, Wikipedia, output จากอุปกรณ์, config) tool แบบ read-only เปลี่ยนผลจาก "model ถูกหลอก" ให้เป็นแค่ "model ตอบผิด" ซึ่งคนจับได้ แต่ tool ที่เขียนได้จะเปลี่ยนเป็นการแก้ไขบนอุปกรณ์จริง มีผลการวัดสองข้อที่กำหนดการออกแบบนี้: thClaws แบบ headless (`-p`) และ OpenClaw (`--local`) รัน MCP tool โดยไม่ถาม และ spec ของ MCP ระบุว่า client ต้องไม่เชื่อ annotation ของ tool ดังนั้นการอนุมัติสำหรับ tool ที่เขียนได้ในอนาคตต้องบังคับใช้ภายใน server ของ tool นั้นเอง (baseline, การอนุมัติที่ผูกกับอุปกรณ์และ payload, verify, audit) ไม่ใช่ฝากไว้กับ agent หรือ prompt

## ระดับ maturity

| ระดับ | หลักฐานที่ต้องมี |
|---|---|
| `experimental` | lint, unit test, smoke test และ security job ใน CI ผ่าน; `risk: read`; validator ทางการผ่าน; scan แล้ว (Cisco Skill Scanner, SkillSpector) โดยมี control ที่ทำงานได้; มี security review อิสระสำหรับทุก skill ที่มี network egress |
| `tested` | ผ่าน experimental + ติดตั้งและใช้งานได้บนทุก agent ที่เราอ้าง + รัน trigger evaluation 3 ครั้งต่อ query, validation ผ่านอย่างน้อย 80% โดยไม่มี false positive + ผ่านกรณี prompt injection อย่างน้อยหนึ่งกรณี |
| `recommended` | ผ่าน tested + ผลประเมิน output ดีกว่า baseline ที่ไม่ใช้ skill ในกรณียาก + ลองกับข้อมูลสังเคราะห์ที่สมจริงแล้วไม่มีข้อผิดพลาดร้ายแรง |

ตอนนี้ยังไม่มี skill ใดสูงกว่า `experimental` ดูระดับได้ด้วย `netpaw list` ตัวเลขและวิธีการอยู่ที่ `docs/READINESS.md`, `docs/evals/TRIGGER-RESULTS.md`

## Trigger evaluation

แต่ละ skill มี query ที่ติด label ไว้ 20 ข้อ (ควร / ไม่ควร trigger แบ่งเป็นชุด train และ validation) `scripts/eval_triggers.py` รัน query เหล่านี้ผ่าน Claude Code ที่โหลด plugin ไว้ แล้วนับว่า tool ของ skill ถูกเรียกหรือไม่ การรันครั้งเดียวต่อ query มี noise (ประมาณบวกลบ 2 query) จึงควรอ่านผลเป็นช่วง

## การประกาศ egress

แต่ละ server ระบุ host ที่อาจติดต่อไว้ใน `mcp/servers.json` (`egress`) lint จะ fail ถ้า server import module ด้านเครือข่ายแต่ไม่ได้ประกาศ host ใดเลย และ `netpaw list` แสดง egress ข้างทุก skill เพื่อให้เห็นว่าข้อมูลอะไรออกจากเครื่องของคุณ
