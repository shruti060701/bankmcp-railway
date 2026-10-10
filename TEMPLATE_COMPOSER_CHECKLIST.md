# Template Composer setup checklist

## Service
- **Image**: `ghcr.io/noskillish/bankmcp:latest` (official, unmodified — no Dockerfile, no custom start command)
- **Icon**: none in the reference; consider the project's own icon if one is published

## Volume
- Mount path: `/data`
- The image's own Dockerfile already sets `ENV DATA_DIR=/data` and `docker-entrypoint.sh` already `chown`s the mount to the unprivileged `node` user before dropping root — no extra permission handling needed, confirmed by reading both files directly

## Networking
- Generate a domain, target port `8080` (matches `ENV PORT=8080` baked into the Dockerfile)
- No `BASE_URL` variable needed — confirmed live in deploy logs: `listening on http://0.0.0.0:8080, public URL https://<the generated domain>` — the app detects its own Railway public domain automatically

## Healthcheck
- Path: `/healthz` — confirmed live: `200 {"ok":true,"version":"...","configured":false}`, no auth required
- **Timeout: `60` seconds** — matches the author's own `railway.json` exactly (the official marketplace template leaves this unset, falling back to Railway's platform default)
- Restart policy: `ON_FAILURE`, max retries **`5`** — matches the author's own `railway.json` (the official marketplace template uses `10`)

## Variables
None required at deploy time, and this is intentional — do not add any. A first-boot setup page (confirmed live, served at `/` with no auth) walks the user through:
1. Registering a free Enable Banking application (the open-banking provider)
2. Generating the app's key pair
3. Setting the admin password

Pre-filling `EB_APP_ID` / `EB_PRIVATE_KEY` / `ADMIN_PASSWORD_HASH` in the Composer would work against this flow — the user doesn't have an Enable Banking application until they register one through the setup page itself.

## Verified live (2026-10-10)
- `GET /healthz` → `200 {"ok":true,"version":"0.1.15","configured":false}`, no auth
- `GET /` → `200`, real setup page (`<title>Set up BankMCP™ · BankMCP™</title>`)
- Deploy logs confirm: `listening on http://0.0.0.0:8080, public URL https://bankmcp-production-37da.up.railway.app` and `not configured yet: open ... to finish setup` listing the three missing setup items — clean, expected first-boot state, not an error
- Deploy reached `SUCCESS` in ~6 seconds

## A note on this template's differentiator
Unlike the other templates in this series, no bug was found in the official `bankmcp` marketplace template — it's built by the project's own author and matches their documented Railway deployment exactly. The only changes here are two settings (`healthcheckTimeout`, `restartPolicyMaxRetries`) tightened to match the author's own `railway.json` precisely, since the marketplace template's values are slightly looser than what the author recommends. This is disclosed rather than inflated into a false "fix."
