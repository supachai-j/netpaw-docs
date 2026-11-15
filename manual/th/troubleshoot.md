# การแก้ปัญหา

| อาการ | สาเหตุที่น่าจะเป็น | วิธีแก้ |
|---|---|---|
| MCP server ขึ้น `failed` ตอนเริ่มครั้งแรก | การเริ่มครั้งแรกต้องดาวน์โหลด package หรือไม่มี `uv` | รัน `uv --version`; ตรวจเครือข่าย; รันคำสั่ง server จาก install snippet ด้วยมือ |
| Codex: `MCP tool call requires approval, but approval policy is never` | tool ไม่มี `readOnlyHint` | รัน `tests/smoke_mcp.py` เพื่อตรวจ annotation; tool ของเราตั้งค่านี้ไว้แล้ว |
| Codex แสดง MCP tool แค่บางส่วนในช่วง session แรก ๆ | ดูเหมือน Codex จะซ่อน server ที่เริ่มช้า; ตอนนี้ทุก server ตอบจาก cache ผ่าน `mcp/_shared/proxy.py` แล้ว แต่ยังไม่ได้วัดผลกับ Codex ซ้ำ | เปิด session เพิ่มอีกสองสามครั้งหรือลองใหม่; รายงานเวอร์ชัน Codex และจำนวน session ที่ต้องใช้ |
| thClaws `mcp spawn refused ... not in the user allowlist` | args หรือ directory เปลี่ยน หรือรันครั้งแรกแบบ headless | เปิด REPL แล้วอนุมัติใหม่ |
| OpenClaw ไม่แสดงรายการ tool โดยตรง | ออกแบบมาแบบนั้น | โมเดลใช้ `tool_search` แล้วตามด้วย `dispatch_action` |
| OpenClaw ติดตั้งเวอร์ชันเก่า | ใช้ Node 22 | ใช้ Node 24.16 ขึ้นไป |
| Hermes: มี `mcp_servers:` ซ้ำ | ถูกต่อท้ายเข้าไปใน block ที่มีอยู่แล้ว | merge ด้วยมือ |
| `subnet_calculator` error `host bits set` | input แบบ `192.0.2.5/26` | ใช้ network address `192.0.2.0/26` |
| `tool list changed upstream; regenerate tools cache` (server ใดก็ได้) | tool list จริงไม่ตรงกับ `mcp/<name>/tools.cache.json` | เป็นการหยุดโดยตั้งใจ; รัน `scripts/gen_tools_cache.py <name>` ตรวจ diff แล้ว commit (จะอัปเดต pin ใน `mcp/servers.json` ด้วย) |
| proxy: `does not match the pinned --expect-sha256` | ไฟล์ cache ถูกแก้โดยไม่ได้อัปเดต pin | แก้แบบเดียวกัน: generate ใหม่แล้วตรวจ diff |
| fwrule: `tool set changed` จาก launcher | ต้นทางเพิ่ม tool ใหม่ | เป็นการหยุดโดยตั้งใจ; รีวิว tool นั้นแล้วอัปเดต `REVIEWED` ใน `mcp/fwrule/launch.py` |
| fwrule: `XML DOCTYPE/ENTITY declarations are not accepted` | payload มี `<!DOCTYPE` หรือ `<!ENTITY` | ลบออก; config อุปกรณ์ปกติไม่จำเป็นต้องมี |
| fwrule: ruleset ถูกปฏิเสธเพราะใหญ่เกิน | proxy จำกัด ruleset ที่ 4 MiB | ลด `FOMC_MAX_RULESET_BYTES` ได้ถ้าต้องการให้เล็กลง; อย่าตั้งเกิน 64 MiB ของต้นทาง |
| rfc-lookup / wikipedia: `response too slow` หรือ error เรื่องขนาด | response ช้าหรือใหญ่เกิน | ลองใหม่ภายหลัง; limit คือ 15 s และไม่กี่ MB ตามที่ออกแบบไว้ |
| lint: `uv run needs --no-project --isolated ...` | entry ของ server ผิดกฎ | ทำตาม pattern ใน [develop](develop.html) |
| lint: `declare egress explicitly` หรือ `import network modules but egress is []` | server ต้องมี entry `egress` | ระบุรายการ host หรือเอา network import ออก |
| `dist/` มี path จากเครื่องอื่น | build ด้วย `--root` ของเครื่องอื่น | build ใหม่ด้วย `--root` ของเครื่องนี้ |

## สิ่งที่ค้างอยู่หลัง uninstall

- thClaws: `netpaw uninstall` ลบ skill และ entry ใน `.thclaws/mcp.json` ของตัวเอง แต่ไม่ได้ลบใน `~/.config/thclaws/mcp_allowlist.json` ของ thClaws เอง ให้ลบบรรทัดของ netpaw ด้วยตัวเอง
- `*.bak-netpaw`: มี backup หนึ่งไฟล์ต่อ config ที่ถูกแก้ (เป็นสถานะก่อน netpaw แก้ครั้งแรก, mode 0600) ไฟล์นี้อาจมี secret จาก config เดิมของคุณ ลบทิ้งเมื่อไม่ต้องใช้แล้ว

ถ้ายังแก้ไม่ได้: เปิด issue พร้อมคำสั่งและ output ตรงตัว ห้ามแนบข้อมูลลูกค้าหรือ credential
