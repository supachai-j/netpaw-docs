# netpaw - documentation site

Source of the public documentation for [netpaw](https://github.com/supachai-j/netpaw): network-engineering skills and MCP tools for any AI agent (Claude Code, Codex, Hermes Agent, OpenClaw, thClaws). Site: https://supachai-j.github.io/netpaw-docs/

**This repo is a mirror.** The source of truth is `docs/manual/` in the main repository; open issues and pull requests there, not here. Pages are English with a Thai section under `manual/th/`.

Build locally: `pip install "markdown==3.10.2" && python3 scripts/build_site.py` -> `site/`. Published by GitHub Pages (Actions) on every push to `main`. Examples use documentation address ranges only (192.0.2.0/24, 2001:db8::/32); no customer data.

License: Apache-2.0.
