# BankMCP™ on Railway

Deploy [BankMCP™](https://github.com/noskillish/bankmcp) — a self-hosted, read-only MCP server that lets Claude, ChatGPT, Cursor, or any MCP client answer questions about your own bank accounts via [Enable Banking](https://enablebanking.com)'s PSD2 open-banking API. No payments, no third party holding your data, single user. MIT-licensed, 277+ stars, actively maintained.

## A different kind of template than usual

Most of the templates in this collection exist because the official Railway template had a real, confirmed bug — a dead environment variable, a healthcheck that silently 401s, a port that doesn't match the domain. This one doesn't. The official `bankmcp` reference template is built by the project's own author, matches their own `railway.json`, and every claim in their README about the Railway deploy ("nothing to fill in," "the server learns its own address from Railway") was verified live and held up exactly as documented. No fake bug has been invented here to justify this template's existence.

What this template does instead: deploys the identical configuration, verified live end-to-end, with two settings tightened to match the author's own `railway.json` exactly — the marketplace template's values are slightly looser than what the author themselves recommends.

| Setting | Official marketplace template | Author's own `railway.json` | This template |
|---|---|---|---|
| `restartPolicyMaxRetries` | `10` | `5` | `5` |
| `healthcheckTimeout` | _(unset — platform default)_ | `60` | `60` |

Neither is a bug fix. They're small alignments with the one source that actually knows this app's intended production behavior: its own author.

## What you get

- The official `ghcr.io/noskillish/bankmcp` image, unmodified
- A persistent volume at `/data` — the app's own Dockerfile already sets `DATA_DIR=/data` and handles volume permissions correctly (it `chown`s the mount and drops root before starting), so this just works
- `/healthz` as the healthcheck — confirmed live to return `{"ok":true,"version":"...","configured":false}` with no auth required
- The app auto-detects its own public Railway domain for OAuth redirects — no `BASE_URL` to set by hand (confirmed live in deploy logs: `listening on http://0.0.0.0:8080, public URL https://...`)

## After deploying

Open the service's public domain. You'll land on a setup page — this is intentional, not a missing-config error. BankMCP™ deliberately ships with zero pre-filled variables because the setup flow walks you through registering a free Enable Banking application (the open-banking provider that connects to your actual bank) and generates your admin password, all in the browser, in about ten minutes. Pre-filling `EB_APP_ID` in the Composer wouldn't help — you don't have one until you register it through this flow.

Full setup instructions: [the project's own README](https://github.com/noskillish/bankmcp#setup).

## License

[MIT](https://github.com/noskillish/bankmcp/blob/main/LICENSE). This template deploys the official image, unmodified.
