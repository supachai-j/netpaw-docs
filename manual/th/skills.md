# รายการ skill

skill ทั้งหกตัวเป็น `risk: read` และ `maturity: experimental` ทุก tool ถูกกำหนด annotation เป็น read-only รับ input แบบเข้มงวด (field ที่ไม่รู้จักจะถูกปฏิเสธ) ไม่ต้องใช้ credential และไม่เปลี่ยนแปลงอุปกรณ์ใด ๆ ดูรายการได้ด้วย `netpaw list`

| Skill | Server | Tools | เครือข่าย |
|---|---|---|---|
| subnet-calculator | `subnet-calc` | 4 | ไม่ออก |
| fwrule-analyzer | `fwrule` | 4 | ไม่ออก |
| rfc-lookup | `rfc-lookup` | 3 | `www.rfc-editor.org` |
| eve-lab-topology-design | `eve-lab` | 1 | ไม่ออก |
| wikipedia-research | `wikipedia` | 5 | `en.wikipedia.org` |
| bgp-registry-intel | `bgp-intel` | 7 | `rpki-validator.ripe.net`, `stat.ripe.net`, `www.peeringdb.com`, `data.iana.org`, RDAP ของ RIR ทั้งห้าแห่ง |

## subnet-calculator (v0.1.0, เขียนเอง)

| Tool | Input | Output |
|---|---|---|
| `subnet_calculator` | `cidr` (IPv4) | network, broadcast, netmask, wildcard, จำนวน host ที่ใช้ได้, first/last host, subnet ก่อนหน้า/ถัดไป, classification, ตัวอย่าง host ไม่เกิน 10 ตัว |
| `subnet_calculator_v6` | `cidr` (IPv6) | ช่วง address, จำนวน /64, ก่อนหน้า/ถัดไป, classification, คำแนะนำการจัดสรร (/64 LAN, /48 site) |
| `subnet_calculator_auto` | `cidr` | เลือก v4 หรือ v6 ให้เอง |
| `same_subnet` | `ip_a`, `ip_b`, `prefix` | ทั้งสองอยู่ subnet เดียวกันหรือไม่ และ subnet ของแต่ละฝั่ง |

input ต้องเป็น canonical network (`address/prefix` ไม่มี host bits ไม่มี zone ของ IPv6 ไม่ใช้รูปแบบ netmask) ถ้าไม่ใช่จะคืน error และไม่เดาให้ classification ที่รู้จัก: documentation (RFC 5737/3849), private (RFC 1918), shared/CGNAT 100.64.0.0/10 (RFC 6598), reserved 240.0.0.0/4, this-network, link-local, loopback, multicast, global

## fwrule-analyzer (v1.1.0, ดัดแปลงจาก NetClaw; engine คือ AutomateIP/fwrule-mcp @ `3bd52d79`)

| Tool | หน้าที่ |
|---|---|
| `parse_policy` | แปลง config ของ vendor ให้เป็น JSON schema กลาง เพื่อตรวจจำนวน rule และ object |
| `analyze_firewall_rule_overlap` | เทียบ rule ที่จะเพิ่มกับ policy เดิมใน 6 มิติ (zone, address, port, protocol, action, application): overlap, shadow, conflict, duplicate |
| `batch_analyze_overlap` | เหมือนกันแต่ตรวจหลาย rule ในครั้งเดียว (รับ JSON ที่ normalise แล้ว) |
| `list_supported_vendors` | vendor และรูปแบบ config ที่รองรับ |

launcher ของเรากำหนดให้ tool เป็น read-only และจะไม่ยอมเริ่มทำงานถ้าชุด tool ของ upstream เปลี่ยนไป มี proxy ที่ตอบ MCP handshake ทันทีจาก cache ที่ตรวจ hash แล้ว และจะเริ่ม server จริงตอนถูกเรียกครั้งแรก (ประมาณหนึ่งวินาที) proxy จะปฏิเสธการประกาศ XML `DOCTYPE`/`ENTITY` ใน argument และจำกัดขนาด ruleset ที่ 4 MiB เพราะ parser ของ PAN-OS ใน upstream สามารถถูกทำให้ขยาย entity ได้ (ดู [safety](safety.html)) ข้อจำกัด: parser ทำงานแบบ best effort, parser XML ของ PAN-OS ยังไม่ได้ fuzz, ควร sanitise config ก่อนใช้

## rfc-lookup (v0.1.0, เขียนเอง)

`search_rfcs` (index ทางการ ใหม่สุดก่อน), `get_rfc` (รายการ section และข้อความทีละหน้า), `get_rfc_section` (ดึงทั้ง section เช่น "Security Considerations") ออก HTTPS ไป `www.rfc-editor.org` เท่านั้น ไม่ตาม redirect, deadline รวม 15 วินาที, จำกัดขนาด, ตรวจเลข RFC และคำค้น, cache ในหน่วยความจำ 16 รายการ output ถูก flag เป็น `untrusted_external_text` และตัดอักขระควบคุมและอักขระที่ซ่อนอยู่ออก ข้อจำกัด: RFC ที่เก่ามากและใช้หัวข้อแปลก ๆ อาจแยก section ไม่ได้ ให้ใช้ `get_rfc` อ่านทีละหน้า

## eve-lab-topology-design (v0.1.0, ขั้นตอนดัดแปลงจาก NetClaw, ตัวตรวจเขียนเอง)

`validate_unl_topology(unl_text, requirements_json, roles_json)` input เป็น**ข้อความ** ไม่ใช่ path ปฏิเสธ `DOCTYPE`/`ENTITY` ก่อน parse; ขีดจำกัด: 2 MB, 500 โหนด, 2000 network, 64 endpoint ต่อ segment รายงานโหนดที่ไม่มีลิงก์ (orphan), ลิงก์ที่ชี้ไปที่ไม่มีอยู่ (dangling), topology ที่ขาดเป็นหลายส่วน, การเชื่อมระหว่าง role ที่ห้าม (ลิงก์ตรงถือว่า fail, ใช้ segment ร่วมกันเป็น warning), single point of failure (ลิงก์ขนานนับว่ามี redundancy), จำนวน uplink และการตรวจตาม requirement role เดาจากชื่อโหนด ไม่เชื่อมต่อ EVE-NG ไม่สตาร์ท lab และไม่ตรวจ image, RAM หรือ routing

## wikipedia-research (v0.1.0, เขียนเอง)

`search_pages`, `get_summary`, `get_content` (แบ่งหน้า), `get_references`, `get_categories` ออก HTTPS ไป `en.wikipedia.org/w/api.php` เท่านั้น เฉพาะ namespace บทความ ไม่ตาม redirect, deadline 15 วินาที, จำกัด 3 MB ผลลัพธ์มี URL ต้นทางและข้อความแจ้งสัญญาอนุญาต CC BY-SA 4.0 และถูก flag ว่า untrusted อย่าใส่ชื่อลูกค้าหรือชื่อภายในในคำค้น เพราะคำค้นถูกส่งไปที่ Wikipedia ไม่มี retry เมื่อโดน rate limit จุดอ่อน: model มักตอบเรื่องพื้นฐานทั่วไปจากความจำโดยไม่เรียก tool นี้

## ที่พิจารณาแล้วและตั้งใจยังไม่ import

| จาก NetClaw | เหตุผล |
|---|---|
| `multivendor-cli` | ตัวกรอง read-only ถูกเลี่ยงได้ด้วย redirection (`redirect`, `tee`, `append`) และมีโหมด `apply_config` |
| `syslog`, ตัวรับ ipfix | เปิด port รอรับโดยไม่มี authentication หรือ TLS |
| `gnmi_set` และทุก tool ที่เขียน config | รอด่านอนุมัติฝั่ง server |
| `nmap` | การ scan ต้องมีการควบคุมขอบเขตก่อน |

การจัดประเภท skill ทั้ง 242 ตัวของ NetClaw อยู่ใน `docs/imports/NETCLAW-TRIAGE.md` และแผนแบ่งเป็นรอบอยู่ใน `docs/imports/NETCLAW-PLAN.md`

## bgp-registry-intel (v0.1.0, เขียนเอง)

`rpki_validate`, `registry_lookup`, `registry_abuse_contact`, `routing_as_overview`, `routing_announced_prefixes`, `peering_network`, `peering_presence` ใช้เฉพาะแหล่งข้อมูลสาธารณะที่ไม่ต้อง authenticate: RPKI validator ของ RIPE NCC, RIPEstat, RDAP ของ RIR ที่รับผิดชอบ (หาผ่าน IANA bootstrap), PeeringDB มี allow-list ของ host และ path ที่ตายตัว ไม่ตาม redirect ยกเว้น RDAP referral ระหว่าง RIR ทั้งห้า (ไม่เกินสอง hop), deadline 15 วินาที, จำกัด 8 MB, ส่ง request ทีละอันโดยเว้น 0.25 วินาทีต่อ host (3.1 วินาทีสำหรับ PeeringDB), cache แยกตามแหล่ง (RPKI 5 นาที, RIPEstat และ PeeringDB 1 ชั่วโมง, RDAP และ bootstrap 24 ชั่วโมง)

- ผล RPKI แยก `valid`, `invalid` (เหตุผล `as` หรือ `length`) และ `not_found` ออกจากกัน ถ้าค้นไม่สำเร็จจะเป็น `validation_unavailable` ไม่ใช่ `not_found`
- prefix และ ASN ที่เป็น private, reserved และ documentation ถูกปฏิเสธในเครื่องก่อนส่ง request ใด ๆ
- คำตอบจาก registry ถูกกรอง: ตัด contact record ของบุคคล (ชื่อ, อีเมล, เบอร์โทร, ที่อยู่) ออก, ไม่แสดง abuse mailbox ที่ลงทะเบียนเป็นของบุคคล และปิดตัวเลขที่ดูเหมือนเบอร์โทรในข้อความอิสระ แต่ field ข้อความอิสระ (ชื่อ object, องค์กร และผู้ถือ) ยังอาจมีชื่อคนอยู่ได้ ไม่คืน contact record ของ PeeringDB เลย (`peering_presence` ขอ PeeringDB ด้วย `depth=2` ซึ่งคำตอบอาจมี contact record มาด้วย แต่จะถูกทิ้ง)
- ข้อจำกัด: ใช้ RPKI validator ตัวเดียว ไม่มีความเห็นที่สอง ถ้ามี referral ไป registry ระดับประเทศ (เช่น registro.br ที่อยู่หลัง LACNIC) จะรายงานไว้แต่ไม่ตามไป prefix ใดที่ซ้อนกับ special-purpose space จะถูกปฏิเสธ ดังนั้นให้ถามด้วย prefix สาธารณะที่เจาะจง ยังไม่ได้ทดสอบกรณีโดน rate limit จากแหล่งข้อมูล ตรวจกับระบบจริงเฉพาะวันที่ 2026-11-15 เท่านั้น
