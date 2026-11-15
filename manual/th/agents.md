# Agent ที่รองรับ

ระดับ: **L1** agent มองเห็น skill, **L2** agent รับ MCP config, **L3** agent สตาร์ท server และแสดงรายการ tool ได้, **L4** model เรียก tool ผ่าน agent และได้ผลลัพธ์กลับมา "Mock" หมายถึง model ปลอมในเครื่องที่ส่ง tool call ตามสคริปต์หนึ่งครั้ง ซึ่งพิสูจน์เส้นทางจาก agent ไป server ได้ แต่ไม่ได้พิสูจน์ว่า LLM จริงจะเลือกใช้ skill

| Agent | เวอร์ชันที่ทดสอบ | ระดับ | Model |
|---|---|---|---|
| Claude Code | 2.1.322 | L1-L4 | จริง |
| Hermes Agent | v0.23.1 | L1-L3 | ยังไม่ได้รัน L4 |
| Codex CLI | 0.169.0 | L1-L4 | mock |
| thClaws | 0.156.0 | L1-L4 | mock |
| OpenClaw | 2026.11.3 | L1-L4 | mock |

ทดสอบบน Ubuntu 24.04 x86_64 ยังไม่ได้ลองบน Windows และ macOS คำสั่งและ output ดิบอยู่ที่ `docs/CONFORMANCE.md` การทดสอบซ้ำภายหลังโดยติดตั้งห้า server พบว่า thClaws และ OpenClaw เรียก wikipedia ได้ถูกต้อง ส่วน Codex ใน `CODEX_HOME` ใหม่ ผู้ทดสอบเห็นเฉพาะ server ที่ตอบทันทีจาก cache (fwrule) ในสี่ session แรก และเห็นครบทั้งห้าตั้งแต่ session ที่ห้าเป็นต้นไป (ทำซ้ำหกครั้ง ผลเหมือนเดิม; รอ 40 วินาทีก็ไม่ช่วย) คำอธิบายที่ใช้อยู่ตอนนี้คือ Codex ซ่อน server ที่สตาร์ทช้า ปัจจุบันทุก server ตอบ `initialize` และ `tools/list` จาก cache ที่ตรวจ hash แล้วผ่าน proxy ที่ใช้ร่วมกัน (วัดในเครื่องได้ประมาณ 0.03 วินาที) และสตาร์ท server จริงตอนถูกเรียกครั้งแรก แต่ยัง**ไม่ได้**วัดซ้ำว่าวิธีนี้แก้อาการบน Codex ได้หรือไม่

## พฤติกรรมเฉพาะของแต่ละ agent ที่ควรรู้

| Agent | หมายเหตุ |
|---|---|
| Claude Code | skill แสดงเป็น `netpaw:<skill>` ส่วน MCP แสดงเป็น `plugin:netpaw:<server>` ถ้าเปิดตัว repo เองจะเห็น server สถานะ `failed` หนึ่งตัวจาก `.mcp.json` ที่ root (ไม่มีผลเสีย) |
| Codex | ชื่อ tool มี namespace เป็น `mcp__<server>` (ขีด `-` กลายเป็น `_`) `exec` ปฏิเสธ tool ที่ไม่มี `readOnlyHint` ตอน cold start อาจไม่เห็น tool ในช่วงแรก ๆ |
| Hermes | ติดตั้งด้วย installer ทางการ ไม่ใช่ pip ถ้า `config.yaml` มี `mcp_servers:` อยู่แล้ว `--with-mcp` จะพิมพ์บรรทัดให้นำไปวางเองแทนการแก้ไฟล์ |
| OpenClaw | ต้องใช้ Node 24.16+ (Node 22 จะติดตั้งเวอร์ชันเก่าโดยไม่แจ้ง) `mcp set` รับ server object ทีละตัว เรียก tool ผ่าน `tool_search` แล้วตามด้วย `dispatch_action` ผลลัพธ์ถูกห่อเป็นเนื้อหาภายนอกที่ไม่น่าเชื่อถือ |
| thClaws | การอนุมัติ stdio server ผูกกับ command line เต็มและ working directory `-p` ไม่รัน slash command และไม่ถามซ้ำ บน Linux `--cli` ต้องมี `libwebkit2gtk-4.1-0 libsoup-3.0-0 libjavascriptcoregtk-4.1-0` |

## Agent อื่น ๆ

อะไรก็ตามที่อ่านโครงสร้าง `SKILL.md` แบบเปิดได้ก็ใช้ข้อความของ skill ได้ และอะไรที่พูด MCP ผ่าน stdio ได้ก็ใช้ server ได้ installer มี target สำหรับ directory ร่วม `.agents/skills`, opencode, Cursor และ Gemini CLI แต่ยัง**ไม่ได้ทดสอบ** และจะถูกปฏิเสธถ้าไม่ใส่ `--allow-untested` ถ้าลองใช้ ให้เปิด PR หรือ issue พร้อมเวอร์ชัน คำสั่งที่ใช้ และสิ่งที่เห็น (ดู [develop](develop.html)) นั่นคือทางที่เร็วที่สุดที่จะย้ายเข้ารายการที่ทดสอบแล้ว
