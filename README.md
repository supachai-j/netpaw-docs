# netpaw — manual (docs only)

Public documentation site for **netpaw** (codename `nxc`): network-engineering skills + MCP tools for Claude Code, Codex, Hermes Agent, OpenClaw and thClaws.
The code lives in a private repository; this repo contains only the Thai manual (`manual/*.md`) and the static-site builder.

Build locally: `pip install "markdown==3.10.2" && python3 scripts/build_site.py` -> `site/`.
Published by GitHub Pages (Actions) on every push to `main`.
Examples use documentation address ranges only (192.0.2.0/24, 2001:db8::/32). No customer data.

License: Apache-2.0.
