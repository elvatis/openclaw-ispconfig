# OpenClaw ISPConfig

OpenClaw plugin for managing ISPConfig through its Remote JSON API. It exposes 292 tools for clients, websites, DNS, mail, databases, FTP, shell users, cron jobs, system settings, and legacy OpenVZ resources.

**Release line:** OpenClaw 2026.8.1
**Distribution:** GitHub and ClawHub only. This package is not published to npm.

Pushing a `v<version>` tag creates a verified GitHub Release with the packaged artifact. It never publishes to npm.

ClawHub releases use GitHub OIDC Trusted Publishing. Start **Publish to ClawHub** manually from the Actions tab. It builds and tests the selected commit, publishes its artifact, and waits for ClawHub security checks. No long-lived ClawHub token is stored in the repository.

## What changed in 2026.8.1

- Updated the manifest and tool adapter for the OpenClaw 2026.8.1 plugin API.
- Registered all tool names through `contracts.tools`.
- Retained full read and write support, with writing disabled by default.
- Added complete guard coverage for the older CRUD tools.
- Removed internal AAHP handoff files and npm publishing automation.
- Removed predictable passwords from the site-provisioning workflow.

## Security model

The plugin is read-only by default. Set `readOnly: false` only when write access is intended.

`allowedOperations` is an optional allowlist. An empty array permits all tools, while a non-empty array permits only the listed tool names. For production, allow only the minimum read and write tools needed for the task.

All API credentials are supplied by environment-variable references. Do not commit a `.env` file, real endpoint, password, API token, host inventory, or deployment notes to this repository.

## Installation

Install from the published ClawHub package after release. For local development, use the OpenClaw plugin installer with a packed local artifact.

## ISPConfig preparation

1. Create a dedicated ISPConfig Remote User.
2. Grant only the Remote API permissions required by the intended tools.
3. Store the endpoint and credentials in the environment used by the OpenClaw Gateway, for example its protected `.env` file. The file must remain local and ignored by Git.
4. Start in read-only mode and enable a narrow allowlist before allowing changes.

## OpenClaw configuration

Use environment references, never literal credentials:

```json
{
  "plugins": {
    "entries": {
      "openclaw-ispconfig": {
        "enabled": true,
        "config": {
          "apiUrl": "${ISPCONFIG_API_URL}",
          "username": "${ISPCONFIG_USER}",
          "password": "${ISPCONFIG_PASS}",
          "readOnly": true,
          "allowedOperations": [
            "isp_sites_list",
            "isp_site_get"
          ],
          "verifySsl": true
        }
      }
    }
  }
}
```

To permit a specific change, set `readOnly` to `false` and add both the required read tools and the specific write tool to `allowedOperations`. Example:

```json
{
  "readOnly": false,
  "allowedOperations": [
    "isp_sites_list",
    "isp_site_get",
    "isp_site_update"
  ]
}
```

An empty `allowedOperations` array allows every registered tool and should be used only for a deliberately unrestricted administrative session.

## Provisioning a site

`isp_provision_site` can create a client, website, DNS zone and records, mail resources, and a database. It is a write operation.

When `createMail` is true, supply `infoMailboxPassword` and `adminMailboxPassword`. When `createDb` is true, supply `databasePassword`. These values must be supplied through a trusted secret-handling path. The plugin does not generate or return default credentials.

## `/ispconfig` quick help

The `/ispconfig` chat command provides an icon-based overview of the active endpoint, plugin version, write-safety state, tool groups, and concise examples. It never displays credentials.

Examples shown by the command include:

- 🧰 `isp_sites_list` to list websites
- 🔎 `isp_site_get` with `primary_id` to inspect a website
- 🌐 `isp_dns_record_list` with `zone_id` to list DNS records
- 🔐 `isp_ssl_status` to inspect certificate status
- ✍️ `isp_site_update` with `client_id`, `primary_id`, and `params` to update a website after write access is explicitly enabled

## Development and verification

```bash
npm ci
npm run build
npm test
clawhub package pack .
clawhub package readiness <artifact>
```

Live tests require `ISPCONFIG_API_URL`, `ISPCONFIG_USER`, and `ISPCONFIG_PASS` in the local environment. They are not part of the standard test suite and must never use a production-changing operation.

## Safety notes

- `readOnly: true` blocks write and provisioning tools before any ISPConfig API request.
- `verifySsl` defaults to `true`.
- The plugin uses the ISPConfig Remote API. Do not bypass it by writing to ISPConfig databases or generated vhost files.
- A dedicated least-privilege Remote User is strongly recommended.

## License

MIT. See [LICENSE](LICENSE).
