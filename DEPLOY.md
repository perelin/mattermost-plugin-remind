# Deployment Runbook — P2Lab Mattermost Server

## Prerequisites

- Go (1.23+) on PATH
- git on PATH
- You are in the repo root of this plugin

## Step 1: Set Environment Variables

```bash
export MM_SERVICESETTINGS_SITEURL=https://mattermost.p2lab.com
export MM_ADMIN_TOKEN=<your admin personal access token>
```

The token is a Mattermost **Admin Personal Access Token** created via:
System Console > Integrations > Personal Access Tokens

## Step 2: Build & Deploy

```bash
make deploy
```

This does:
1. Builds Go server binaries (multi-arch: linux, darwin, windows)
2. Bundles everything into `dist/<plugin-id>-<version>.tar.gz`
3. Uploads the bundle to the server via UploadPluginForced API
4. Enables the plugin via EnablePlugin API

## Step 3: Verify

```bash
# Check plugin status via API
curl -s -H "Authorization: Bearer $MM_ADMIN_TOKEN" \
  "$MM_SERVICESETTINGS_SITEURL/api/v4/plugins" | python3 -m json.tool
```

The plugin should appear as active under System Console > Plugins > Plugin Management.

## Useful Commands

| Command            | What it does                        |
|--------------------|--------------------------------------|
| `make deploy`      | Full build + upload + enable         |
| `make dist`        | Build + bundle only (no upload)      |
| `make server`      | Build server binaries only           |
| `make test`        | Run Go tests                         |
| `make reset`       | Disable + re-enable (restart plugin) |
| `make disable`     | Disable the plugin                   |
| `make enable`      | Enable the plugin                    |

## How Authentication Works

The `build/pluginctl` tool connects in this order:

1. Unix socket (local mode) — only works if Mattermost runs on the same machine
2. `MM_ADMIN_TOKEN` — personal access token with admin privileges (used here)
3. `MM_ADMIN_USERNAME` + `MM_ADMIN_PASSWORD` — username/password fallback

## Troubleshooting

| Error                               | Cause / Fix                                                    |
|-------------------------------------|----------------------------------------------------------------|
| `MM_SERVICESETTINGS_SITEURL is not set` | Env vars not exported — run `export` commands first          |
| `failed to upload plugin bundle`    | Plugin uploads disabled in System Console, or token lacks admin rights |
| `failed to enable plugin`           | Plugin crashed on startup — check server logs                  |
| Build fails on Go                   | Check Go version matches `go.mod` requirement                   |

## Notes

- Plugin ID and version are read from `plugin.json`
- Min Mattermost server version is specified in `plugin.json` (currently v11.0.0)
- Credentials are passed via environment variables only — never commit secrets to the repo
