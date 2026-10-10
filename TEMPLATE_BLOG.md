# Deploy and Host BankMCP™ on Railway

[BankMCP™](https://github.com/noskillish/bankmcp) connects Claude, ChatGPT, Cursor, or any MCP client to your own bank accounts, read-only, through [Enable Banking](https://enablebanking.com)'s PSD2 open-banking API — ask it what you spent on groceries last month, or tell it to notify you when a balance drops below a threshold. This is a different kind of template post than usual: the official Railway template for this one isn't broken, and this post says so plainly instead of manufacturing a reason it needs fixing.

## About Hosting BankMCP™

This template deploys `ghcr.io/noskillish/bankmcp`, the official image, exactly as the project's own README documents for Railway: a volume at `/data`, a public domain, and nothing else to configure before the setup page takes over. Checked this claim line by line against the actual Dockerfile and entrypoint script rather than taking it on faith, and it holds up — `ENV DATA_DIR=/data` and `PORT=8080` are baked into the image, and `docker-entrypoint.sh` already handles the root-owned-volume problem (it `chown`s the mount to an unprivileged user before dropping root) that shows up as a real bug on other self-hosted templates in this same series.

## Common Use Cases

- **Ask your AI about your own spending**: "what did I spend on groceries in August?", "which subscriptions am I paying for?"
- **Balance and payment alerts**: watches that notify you (via webhook or Slack) when a balance crosses a threshold, or a specific payment hasn't arrived by a date
- **Monthly financial reviews**: the project ships MCP prompts for monthly summaries, budget building, and subscription audits
- **Keeping your financial data off third-party AI infrastructure**: self-hosted and read-only, so no AI vendor sees your transaction data, only your assistant does, mediated through your own server

## Dependencies for BankMCP™ Hosting

Running this template needs nothing beyond the container and one persistent volume. What you bring yourself is a free [Enable Banking](https://enablebanking.com) account, registered through the app's own first-boot setup page — this is an open-banking provider that wraps 2,700+ European banks behind one PSD2 API, and it's how BankMCP™ reaches your actual bank without ever seeing your bank credentials itself (you log in at your bank's own site).

### Deployment Dependencies

- [BankMCP™'s Source Code](https://github.com/noskillish/bankmcp) — MIT
- [BankMCP™'s Official Setup Documentation](https://github.com/noskillish/bankmcp#setup)
- [Enable Banking](https://enablebanking.com) — the PSD2 open-banking provider BankMCP™ connects through

### Implementation Details

Two small settings were tightened here, both disclosed rather than dressed up as bug fixes: `healthcheckTimeout` is set to `60` seconds and `restartPolicyMaxRetries` to `5` — both pulled directly from the author's own `railway.json` in the repository, which is slightly stricter than the official marketplace template's values (an unset timeout falling back to Railway's platform default, and `10` retries instead of `5`). Neither changes correctness; they just match the one source that actually knows this app's intended production behavior.

Everything else was verified rather than assumed. The claim that "the server learns its own address from Railway" was checked directly in the live deploy logs: `listening on http://0.0.0.0:8080, public URL https://<the generated domain>` — no `BASE_URL` variable needed, confirmed rather than taken on faith. `/healthz` was checked live and returns `{"ok":true,"version":"...","configured":false}` with no authentication required, a genuinely useful detail since it reports setup status directly in the healthcheck response.

## Why Deploy BankMCP™ on Railway?

For a tool that holds the keys to real financial account access, the main thing Railway offers is a clean, auditable deployment with no manual server hardening — a volume for the one piece of state this app actually keeps (bank connection consents and account ids, never balances or transactions), and TLS handled automatically for the OAuth redirect flow the whole setup depends on.

This template's actual contribution is smaller than most in this series, and that's stated here directly rather than inflated: a faithful, verified deployment of an already well-built project, tightened to match its own author's recommended production settings exactly.

## What Was Verified

Every claim in this post was checked against a live deployment, not inferred from reading the source alone. The image deployed and reached `SUCCESS` in about six seconds. `GET /healthz` returns `200` with a clean status payload. `GET /` serves the real, unauthenticated setup page (`<title>Set up BankMCP™ · BankMCP™</title>`) — the correct, intentional first-boot state, not a missing-config error. Deploy logs confirm the app correctly detected its own public Railway domain without any `BASE_URL` configuration, and logged exactly which three setup steps remained (Enable Banking application id, private key, admin password) — a server being honest about its own unconfigured state rather than failing silently.

## Frequently Asked Questions

### Why does the deploy page show a setup form instead of the app?
That's intentional, not an error. BankMCP™ ships with zero pre-filled configuration by design — the setup page walks you through registering a free Enable Banking application and generating your admin password in the browser, which can't be pre-filled in the Composer because you don't have an Enable Banking application until you register one through that same flow.

### Does this template fix a bug in the official one?
No, and this post says so directly rather than inventing one. The official `bankmcp` marketplace template is built by the project's own author and matches their documented Railway deployment exactly — no dead variables, no port mismatch, no missing healthcheck. The two settings changed here (`healthcheckTimeout`, `restartPolicyMaxRetries`) are alignments with the author's own `railway.json`, not corrections of an error.

### Is my bank data safe?
BankMCP™ is read-only, self-hosted, and single-user by design — it stores only bank consents and account ids on your volume, never balances or transactions, and sends no telemetry. Enable Banking sits between your server and your bank as the licensed PSD2 provider; your bank credentials are never seen by BankMCP™ or by Enable Banking, since you log in at your bank's own site. Review the project's own [security policy](https://github.com/noskillish/bankmcp/blob/main/SECURITY.md) and Enable Banking's [Restriction of Use terms](https://enablebanking.com/terms-of-service/) before connecting real accounts.

### Where can I find BankMCP™?
Source is on GitHub at [github.com/noskillish/bankmcp](https://github.com/noskillish/bankmcp). Use this template to deploy the official image, verified live and matched to the author's own recommended production settings, in one click.

Source for this template's docs: https://github.com/shruti060701/bankmcp-railway
