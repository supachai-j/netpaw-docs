# Roadmap

เรียงตามหลักฐาน ไม่ใช่ตามความอยาก รายการแบบละเอียดที่ตรวจสอบได้อยู่ใน `docs/ROADMAP.md`

1. **ทำให้ skill ที่มีอยู่มีประโยชน์แบบวัดผลได้** ประเมิน trigger 3 รอบต่อ query ในทุก skill และประเมิน output เทียบระหว่างมีกับไม่มี skill (ทำแล้ว: ตอนนี้ทุก server เริ่มทำงานหลัง lazy proxy และในการทดสอบ Codex แสดง server ครบทุกตัวตั้งแต่ session แรก)
2. **รันกับ model จริงในแต่ละ agent** (ต้องมีคน login เข้า agent บนเครื่องทดสอบ) ระหว่างนี้ agent ทุกตัวยกเว้น Claude Code ยังมีหลักฐานแค่ระดับ mock model
3. **ความเรียบร้อยของ project:** ใช้ install directory ที่คงที่สำหรับ MCP config แทน checkout, fuzz XML parser ของ fwrule, สแกน skill ตามรอบเวลาใน CI, ตั้ง timeout ของ proxy ต่อการเรียกแต่ละครั้ง และจำกัดขนาด cache เป็นจำนวน byte (ทำแล้ว: cache ของ tool definition ถูก pin ด้วย hash สำหรับทุก server)
4. **เพิ่ม skill แบบ read-only** จาก NetClaw เป็นระลอก: tool แบบ offline, การค้นข้อมูลบนอินเทอร์เน็ตสาธารณะ (ตัวแรกทำแล้ว: bgp-registry-intel) จากนั้นจึงเป็นการเข้าถึงอุปกรณ์แบบ read-only ด้วย credential
5. **Skill ที่เขียนข้อมูล** จะทำหลังจากมี gate ฝั่ง server แล้วเท่านั้น: change request, baseline, การอนุมัติที่ผูกกับอุปกรณ์และ payload, การ verify, audit พร้อม test กับ lab
6. **เพิ่ม agent:** ติดตั้งแบบทดสอบแล้วสำหรับ `.agents/skills`, opencode, Cursor, Gemini CLI รวมถึง Windows และ macOS

ไม่มีในแผน: tool ที่เปิด port รอรับการเชื่อมต่อ รับข้อความ CLI อะไรก็ได้ หรือเขียน configuration โดยไม่มี gate
