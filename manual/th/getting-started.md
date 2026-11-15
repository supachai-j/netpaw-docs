# เริ่มต้นใช้งาน

## สิ่งที่ต้องมี

- Python 3 (ทดสอบกับ 3.12) และ [`uv`](https://docs.astral.sh/uv/) MCP server จะถูกเปิดด้วย `uv run --no-project --isolated` ดังนั้นการเริ่มครั้งแรกจะดาวน์โหลด package ตามเวอร์ชันที่ pin ไว้
- git และ agent ที่รองรับอย่างน้อยหนึ่งตัว: Claude Code, Codex CLI, Hermes Agent, OpenClaw (ต้องใช้ Node 24.16 ขึ้นไป) หรือ thClaws
- Linux ทดสอบแล้วเฉพาะ Ubuntu 24.04 x86_64

## ติดตั้งด้วย netpaw CLI

```bash
git clone https://github.com/supachai-j/netpaw.git ~/netpaw && cd ~/netpaw
bin/netpaw doctor      # uv, python, detected agents
bin/netpaw agents      # supported agents and whether each is tested
bin/netpaw profiles    # minimal | standard | online | full
bin/netpaw list        # skills with risk, maturity and network egress
```

ลอง dry run ก่อน (เป็นค่าเริ่มต้น) แล้วค่อย apply:

```bash
bin/netpaw install -a codex hermes -p standard --root "$PWD"
bin/netpaw install -a codex -p standard --root "$PWD" --with-mcp --apply
bin/netpaw uninstall --apply
```

- **ค่าเริ่มต้นคือ dry run** จะไม่มีการเขียนไฟล์ใดถ้าไม่ใส่ `--apply`
- `--with-mcp` จะแก้ config ของ agent เองสำหรับ Codex (`config.toml`), Hermes (`config.yaml`) และ thClaws (`.thclaws/mcp.json` ของ project) โดยเก็บ backup ไว้ไฟล์ละหนึ่งชุด (`*.bak-netpaw`, mode 0600) และเพิ่มเฉพาะ entry ที่ตัวเองเป็นเจ้าของ backup อาจมี secret จาก config เดิมของคุณอยู่ ใช้งานเสร็จแล้วให้ลบทิ้ง
- manifest ที่ `~/.netpaw/manifest.json` บันทึก hash ของทุกไฟล์ที่ติดตั้ง และ `uninstall` จะไม่ลบไฟล์ที่คุณแก้ไขเอง
- Profile: `minimal` (subnet-calculator), `standard` (+ fwrule-analyzer, eve-lab-topology-design), `online` (เพิ่ม rfc-lookup, wikipedia-research และ bgp-registry-intel ซึ่งใช้อินเทอร์เน็ต), `full` (ทั้งหมด) เพิ่ม skill ทีละตัวได้ด้วย `-s <skill>`
- Agent ที่ installer รู้จักแต่เรา **ยังไม่ได้** ทดสอบ (`.agents/skills`, opencode, Cursor, Gemini CLI) ต้องใส่ `--allow-untested`
- ไม่มี `pip install netpaw` หรือ `npx netpaw` อย่าติดตั้ง package ที่ใช้ชื่อนี้

## หมายเหตุแยกตาม agent

**Claude Code** - โหลด plugin จาก clone:

```bash
claude --plugin-dir ~/netpaw
```

คุณควรเห็น skill ในชื่อ `netpaw:<skill>` และ MCP server เชื่อมต่ออยู่ ช่องทาง marketplace (`/plugin marketplace add supachai-j/netpaw`, `/plugin install netpaw`) ยังไม่ได้ทดสอบ ถ้าเปิด Claude Code ภายใน repo นี้เอง จะมี server หนึ่งตัวขึ้นสถานะ `failed` เพราะ `.mcp.json` ที่ root ถูกอ่านเป็นไฟล์ของ project แต่ไม่กระทบสำเนาที่อยู่ใน plugin

**Codex CLI** - `netpaw install -a codex ... --with-mcp --apply` Codex เรียก tool ตาม namespace (`mcp__subnet_calc`) แค่บอกเป้าหมายที่ต้องการก็พอ tool แบบ read-only ของเรามี `readOnlyHint` ซึ่ง `codex exec` บังคับให้มี ตอน cold start Codex อาจแสดง server เพียงบางตัวในช่วงแรก ๆ ให้ลองใหม่

**Hermes Agent** - `netpaw install -a hermes ... --with-mcp --apply` แล้วรัน `hermes skills list` และ `hermes mcp test subnet-calc` ต้องติดตั้ง Hermes ด้วย installer ทางการ ไม่ใช่ pip

**OpenClaw** - ติดตั้ง skill ด้วย `openclaw skills install dist/openclaw/skills/<name>` และเพิ่ม server ทีละตัวด้วย `openclaw mcp set <name> '<json>'` (ใส่ server object เดียว ไม่ใช่ทั้งบล็อก `mcpServers`) model จะเข้าถึง MCP tool ผ่าน `tool_search` แล้วตามด้วย `dispatch_action`

**thClaws** - ระดับ project: `netpaw install -a thclaws ... --project --with-mcp --apply` thClaws จะให้คุณอนุมัติ stdio server แต่ละตัวหนึ่งครั้ง โหมด `-p` (headless) จะไม่ถามซ้ำเมื่อมีการเรียก tool

วิธีติดตั้งเองโดยไม่ใช้ CLI: `python3 scripts/build.py --root "$PWD"` จะเขียนไฟล์เฉพาะของแต่ละ agent ไว้ที่ `dist/` แล้ว copy ไปตามที่อธิบายไว้ใน `docs/RUNTIME-MATRIX.md`

## ตรวจว่าใช้งานได้

ลองถาม: *"192.0.2.10 กับ 192.0.2.70 อยู่ใน /26 เดียวกันไหม?"* agent ควรเรียก `same_subnet` และตอบว่า **ไม่** (192.0.2.0/26 และ 192.0.2.64/26) ถ้าตอบโดยไม่เรียก tool คำตอบอาจยังถูก แต่คุณยังไม่ได้ยืนยันว่า skill เชื่อมต่อแล้ว ให้ดูที่ log การเรียก tool ของ agent
