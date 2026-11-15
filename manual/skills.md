# Skill catalog

All six skills are `risk: read` and `maturity: experimental`. Every tool is annotated read-only, takes strict input (unknown fields are rejected), needs no credentials and never changes a device. List them with `netpaw list`.

| Skill | Server | Tools | Network |
|---|---|---|---|
| subnet-calculator | `subnet-calc` | 4 | none |
| fwrule-analyzer | `fwrule` | 4 | none |
| rfc-lookup | `rfc-lookup` | 3 | `www.rfc-editor.org` |
| eve-lab-topology-design | `eve-lab` | 1 | none |
| wikipedia-research | `wikipedia` | 5 | `en.wikipedia.org` |
| bgp-registry-intel | `bgp-intel` | 7 | `rpki-validator.ripe.net`, `stat.ripe.net`, `www.peeringdb.com`, `data.iana.org`, RDAP of the five RIRs |

## subnet-calculator (v0.1.0, original)

| Tool | Input | Output |
|---|---|---|
| `subnet_calculator` | `cidr` (IPv4) | network, broadcast, netmask, wildcard, usable hosts, first/last host, previous/next subnet, classification, up to 10 sample hosts |
| `subnet_calculator_v6` | `cidr` (IPv6) | range, number of /64s, previous/next, classification, allocation hint (/64 LAN, /48 site) |
| `subnet_calculator_auto` | `cidr` | picks v4 or v6 |
| `same_subnet` | `ip_a`, `ip_b`, `prefix` | whether both are in one subnet, and each side's subnet |

Input must be a canonical network (`address/prefix`, no host bits, no IPv6 zone, no netmask form); otherwise an error is returned and nothing is guessed. Known classes: documentation (RFC 5737/3849), private (RFC 1918), shared/CGNAT 100.64.0.0/10 (RFC 6598), reserved 240.0.0.0/4, this-network, link-local, loopback, multicast, global.

## fwrule-analyzer (v1.1.0, adapted from NetClaw; engine is AutomateIP/fwrule-mcp @ `3bd52d79`)

| Tool | Purpose |
|---|---|
| `parse_policy` | Normalise a vendor configuration to a common JSON schema so you can check rule and object counts |
| `analyze_firewall_rule_overlap` | Compare a candidate rule with an existing policy on six dimensions (zone, address, port, protocol, action, application): overlap, shadow, conflict, duplicate |
| `batch_analyze_overlap` | Same for many candidates at once (normalised JSON input) |
| `list_supported_vendors` | Supported vendors and config formats |

Our launcher marks the tools read-only and refuses to start if the upstream tool set changes. A proxy answers the MCP handshake instantly from a hash-checked cache and starts the real server on the first call (about one second). It rejects XML `DOCTYPE`/`ENTITY` declarations in arguments and caps the ruleset at 4 MiB because the upstream PAN-OS parser can be made to expand entities (see [safety](safety.html)). Limits: parsers are best effort; PAN-OS XML parser not fuzzed; sanitise configs before use.

## rfc-lookup (v0.1.0, original implementation)

`search_rfcs` (official index, newest first), `get_rfc` (section list and text by page), `get_rfc_section` (one whole section, e.g. "Security Considerations"). Only HTTPS to `www.rfc-editor.org`; redirects are refused; 15 s overall deadline; size caps; RFC numbers and queries are validated; 16-entry in-memory cache. Output is flagged `untrusted_external_text` and stripped of control and hidden characters. Limit: very old RFCs with unusual headings may not split into sections; use `get_rfc` page by page.

## eve-lab-topology-design (v0.1.0, workflow adapted from NetClaw, validator original)

`validate_unl_topology(unl_text, requirements_json, roles_json)`. Input is **text**, never a path. `DOCTYPE`/`ENTITY` are refused before parsing; limits: 2 MB, 500 nodes, 2000 networks, 64 endpoints per segment. Reports orphan nodes, dangling links, partitions, forbidden role adjacencies (direct links fail, shared segments warn), single points of failure (parallel links count as redundant), uplink counts and requirement checks. Roles are inferred from node names. It does not connect to EVE-NG, start labs or inspect images, RAM or routing.

## wikipedia-research (v0.1.0, original implementation)

`search_pages`, `get_summary`, `get_content` (paged), `get_references`, `get_categories`. Only HTTPS to `en.wikipedia.org/w/api.php`; article namespace only; redirects refused; 15 s deadline; 3 MB cap. Results include the source URL and the CC BY-SA 4.0 notice and are flagged untrusted. Do not put customer or internal names in queries; they go to Wikipedia. No retry on rate limiting. Weak spot: models often answer general background from memory without calling it.

## Considered and deliberately not imported (yet)

| From NetClaw | Why |
|---|---|
| `multivendor-cli` | read-only filter can be bypassed with redirection (`redirect`, `tee`, `append`); has an `apply_config` mode |
| `syslog`, ipfix receivers | open a listening port without authentication or TLS |
| `gnmi_set` and every config-writing tool | wait for a server-side approval gate |
| `nmap` | scanning needs scope control first |

The classification of all 242 NetClaw skills is in `docs/imports/NETCLAW-TRIAGE.md` and the plan by wave in `docs/imports/NETCLAW-PLAN.md`.

## bgp-registry-intel (v0.1.0, original implementation)

`rpki_validate`, `registry_lookup`, `registry_abuse_contact`, `routing_as_overview`, `routing_announced_prefixes`, `peering_network`, `peering_presence`. Public, unauthenticated sources only: RIPE NCC RPKI validator, RIPEstat, RDAP at the responsible RIR (found through the IANA bootstrap), PeeringDB. Fixed host and path allow-list; redirects refused except an RDAP referral between the five RIRs (two hops at most); 15 s deadline; 8 MB cap; serial with 0.25 s between requests per host (3.1 s for PeeringDB); cache per source (RPKI 5 min, RIPEstat and PeeringDB 1 h, RDAP and bootstrap 24 h).

- RPKI results keep `valid`, `invalid` (reason `as` or `length`) and `not_found` apart; a failed lookup is `validation_unavailable`, never `not_found`.
- Private, reserved and documentation prefixes and ASNs are refused locally before any request.
- Registry answers are filtered: contact records of individuals (name, email, phone, address) are dropped, abuse mailboxes registered to an individual are withheld, and phone-like numbers in free text are masked. Free-text fields (object, organisation and holder names) can still contain a person's name. PeeringDB contact records are never returned (`peering_presence` asks PeeringDB for `depth=2`, whose answer can include them; they are discarded).
- Limits: one RPKI validator, no second opinion. A referral to a national registry (for example registro.br behind LACNIC) is reported, not followed. Any prefix that overlaps special-purpose space is refused, so ask about the specific public prefix. Not tested against rate limiting by the sources. Verified live on 2026-11-15 only.
