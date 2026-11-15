# การพัฒนา

## โครงสร้าง

```
skills/<name>/SKILL.md      source skill (agent-neutral), evals/ with trigger and output cases
mcp/servers.json            server registry ({root} = install path); egress and capability per server
mcp/<server>/               our server code, or launcher + hash-locked requirements for an upstream one
mcp/_shared/                shared helpers (strict input, hidden-character stripper)
profiles.json               minimal | standard | online | full
imports/netclaw.json        pinned imports from NetClaw
scripts/                    build, lint, import, eval and cache tools
tests/                      unit, security regression, smoke MCP, runtime scripts
docs/                       plan, decisions (ADR style), reviews, conformance, manual
```

## Frontmatter ของ skill

```yaml
---
name: my-skill            # equals the folder name, lowercase and dashes, max 64
description: "Use this skill when ... because ..."   # when to use, what it does; no ': ' or ' #' in plain scalars
license: Apache-2.0
compatibility: Requires uv. Network egress to example.org only.
metadata:
  version: "0.1.0"
  risk: read              # read | write
  maturity: experimental  # experimental | tested | recommended
  mcp_servers: "my-server"
  requires_bins: "uv"
---
```

ระดับบนสุดอนุญาตเฉพาะ `name, description, license, compatibility, metadata` และห้ามใช้ `allowed-tools` ไฟล์ที่วางข้าง `SKILL.md` จำกัดไว้แค่ `references/*.md`, `agents/*.yaml` และ `evals/*.json` ให้เขียนชื่อ tool ตรง ๆ ไม่ต้องมี prefix ของ agent skill ที่เป็น `risk: write` ต้องมี `metadata.approval: required` และหัวข้อ `## Approval` แต่ gate ตัวจริงต้องอยู่ใน server

## เพิ่ม MCP server

1. ใช้ Python stdlib เป็นหลัก ใช้ FastMCP เฉพาะส่วน wrapper และเรียก `harden` เพื่อให้ปฏิเสธ field ที่ไม่รู้จัก read tool ใช้ `ToolAnnotations(readOnlyHint=True, destructiveHint=False, idempotentHint=True, openWorldHint=False)` ส่วน write tool ห้ามตั้ง `readOnlyHint`
2. ลงทะเบียนใน `mcp/servers.json` พร้อม `capability` และรายการ `egress` ที่ระบุชัดเจน (ใส่ `[]` ถ้าไม่มี) ให้เริ่ม server ผ่าน shared proxy: `python3 {root}/mcp/_shared/proxy.py --expect-sha256 <hash> --cache {root}/mcp/<name>/tools.cache.json -- uv run --quiet --no-project --isolated --with mcp==<pin> python {root}/mcp/<name>/server.py`
3. สร้าง cache ของนิยาม tool และ pin: `python3 scripts/gen_tools_cache.py <name>` (เขียน `mcp/<name>/tools.cache.json` และ hash ลงใน `servers.json`) ตรวจ diff ทุกครั้ง นิยาม tool ที่เปลี่ยนต้องผ่านการรีวิว ห้ามยอมรับอัตโนมัติ
4. เพิ่ม test รวมถึง input ที่มุ่งร้าย (ขนาดเกิน, รูปแบบผิด, อักขระซ่อน, ข้อความ injection ในข้อมูลที่ดึงมา)

## ภาษา

Python เป็นภาษาหลักสำหรับ server ใหม่ Go เป็นภาษาที่สอง ใช้เมื่อ Python ไม่เหมาะ (เช่น gNMI หรือต้องการ binary เดียวโดยไม่ใช้ `uv`) TypeScript และ Rust ยังไม่เปิดให้ใช้ การตัดสินใจและเงื่อนไขอยู่ที่ `docs/DECISIONS.md` D-13

server ที่เขียนด้วยภาษาอื่นต้องประกาศ `language` ใน `mcp/servers.json` และผ่าน `scripts/langpolicy.py` (ตรวจโดย `scripts/lint.py`): มี lockfile ที่ commit แล้ว (`go.sum`, `Cargo.lock`, `package-lock.json`); command ต้องเป็น artefact ที่ build แล้ว (`{root}/mcp/<name>/bin/<name>` หรือ `node` กับไฟล์ใต้ `{root}/mcp/<name>/dist/`) ห้ามใช้ `go run`, `cargo run`, `npx` หรือ `npm exec`; มี `egress` ที่ระบุชัดเจน; ไม่ spawn process; และมี `release_checks` ที่ระบุชื่อ CI job ที่ build, hash และสแกน server นั้น การตรวจเหล่านี้เป็นแค่ tripwire แบบ static server ตัวแรกที่เป็นภาษาอื่นต้องมี security review อิสระและ runtime test บนทุก agent ด้วย ปัจจุบันยังไม่มี server ที่ไม่ใช่ Python

## Dev loop

```bash
bin/netpaw lint && bin/netpaw test && bin/netpaw smoke
NETPAW_LIVE_TESTS=1 python3 -m unittest discover -s tests   # also live checks against public hosts
python3 scripts/gen_tools_cache.py --all --check             # tool definitions unchanged
claude plugin validate .
```

security tool ที่ใช้ใน CI: gitleaks, bandit, semgrep, pip-audit, zizmor, `skills-ref` รันในเครื่องได้โดยใช้เวอร์ชันที่ pin ไว้ใน `.github/workflows/security.yml`

## ประเมิน skill

- ประเมินการ trigger: `python3 scripts/eval_triggers.py --runs 3 --out <file> <skill>` (ใช้ Claude Code และโควตาโมเดลของคุณ แจ้งผู้รีวิวด้วยว่าเรียกไปกี่ครั้ง)
- ประเมิน output: `scripts/eval_outputs.py` เทียบคำตอบแบบมีและไม่มี skill ด้วยการตรวจแบบ deterministic
- บันทึกสิ่งที่รันและ output ไว้ใน `docs/evals/` อย่าเขียนตัวเลขที่ไม่ได้วัดจริง

## Import skill จาก NetClaw

```bash
python3 scripts/import-netclaw.py --upstream <checkout at the pinned commit>
```

script จะตรวจ commit, เอาเฉพาะ `SKILL.md`, ทำการแก้ไขแบบ exact-match (fail ถ้าต้นทางเปลี่ยน), เขียน frontmatter ใหม่, บันทึก hash และอัปเดต `NOTICE` ทุกการ import ต้องมีบันทึกรีวิวใน `docs/imports/<skill>.md` (ต้นทางทำอะไร, เราเปลี่ยนอะไร, อะไรที่เราไม่ได้ตรวจ) และตรวจ licence ด้วย ลำดับงานอยู่ใน `docs/imports/NETCLAW-PLAN.md`

## รายงานผล runtime test

ระบุ agent และเวอร์ชัน, OS, คำสั่งที่ใช้ตรงตัว, ใช้โมเดลจริงหรือ mock, ระดับที่ไปถึง (L1-L4) และ output ดิบ บอกด้วยว่าอะไรที่ไม่ได้ทดสอบ และลบ test install ออกหลังทดสอบเสร็จ

## กระบวนการ

บทบาท, gate และกฎที่ว่าผู้เขียนต้องไม่เป็นผู้รีวิวด้านความปลอดภัยเพียงคนเดียว อยู่ที่ `docs/AI-SDLC.md` การตัดสินใจมีหมายเลขกำกับใน `docs/DECISIONS.md` กฎการมีส่วนร่วมอยู่ที่ `CONTRIBUTING.md`
