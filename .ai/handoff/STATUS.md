# STATUS - openclaw-ispconfig

## Current Version: 0.4.0

- **npm:** @elvatis_com/openclaw-ispconfig@0.4.0
- **ClawHub:** openclaw-ispconfig@0.4.0
- **GitHub:** https://github.com/elvatis/openclaw-ispconfig/releases/tag/v0.4.0

## Build Health
- TypeScript strict build: OK (clean compile)
- 65 tools registered with JSON Schema parameters
- Plugin API uses `execute()` (not `run()`)
- openclaw.plugin.json: version synced
- Live integration tested against isp.elvatis.com

## What Changed in v0.4.0 (2026-03-16)
**Feature:** 14 new tools + expanded DNS record types. Full CRUD for all resources.

New tools:
- isp_dns_zone_get, isp_dns_zone_update
- isp_mail_domain_update, isp_mail_user_update, isp_mail_alias_update, isp_mail_forward_update
- isp_db_get, isp_db_update, isp_db_user_get, isp_db_user_update
- isp_ftp_user_get, isp_ftp_user_update, isp_shell_user_get, isp_shell_user_update

DNS record type expansion:
- isp_dns_record_add/update/delete now support 9 types: A, AAAA, MX, TXT, CNAME, SRV, CAA, NS, PTR (was 5)

Other changes:
- Extended KNOWN_METHODS with 26 additional API methods
- Added validation schemas for all new tools
- Updated /ispconfig command to reflect 65 tools
- Updated dnsMethodForType() to support SRV, CAA, NS, PTR

## Plugin API Contract (IMPORTANT)
OpenClaw expects tools registered via `api.registerTool()` to have:
```ts
api.registerTool({
  name: "tool_name",
  description: "...",
  parameters: { type: "object", properties: { ... }, required: [...] },
  execute: (params) => tool.run(params),  // NOT "run" - must be "execute"
});
```
- `parameters` MUST always be present with at least `{ type: "object", properties: {} }`
- `execute` is the function name, not `run`
- Missing `parameters` causes UI crash on `.properties` access
- Using `run` instead of `execute` causes `tool.execute is not a function`

## Tools (65 total)
- 22 read tools (system_info, client_list/get, sites_list/get, dns_zone_list/get, db_get, db_user_get, ftp_user_get, shell_user_get, etc.)
- 42 write tools (full CRUD for clients, sites, dns zones, dns records [9 types], mail domains/users/aliases/forwards, databases/users, ftp/shell users, cron)
- 1 provisioning (provision_site - full stack in one call)

## Architecture
- Session-based ISPConfig JSON API client with auto-reconnect
- Safety guards via `readOnly` and `allowedOperations` config
- Validation layer (src/validate.ts) with per-tool schemas
- Connected to: isp.elvatis.com:8080
