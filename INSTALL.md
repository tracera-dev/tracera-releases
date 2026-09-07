# Tracera on-prem install

Artifacts and release notes: [GitHub Releases](https://github.com/tracera-dev/tracera-releases/releases).

## Requirements

- Docker Engine + Compose v2
- From about **4 GB RAM** per instance (roughly **1–2 projects**, **1–5 plans**, **1–3 thousand** test cases)
- Outbound HTTPS to `ghcr.io` for image pull

## 1. Image registry login

Tracera support sends a **pull token** (one per customer). Use it to log in:

**bash / zsh / Git Bash / WSL:**

```bash
export TRACERA_GHCR_TOKEN='ghp_…'
echo "$TRACERA_GHCR_TOKEN" | docker login ghcr.io -u github --password-stdin
```

**PowerShell:**

```powershell
$env:TRACERA_GHCR_TOKEN = 'ghp_…'
$env:TRACERA_GHCR_TOKEN | docker login ghcr.io -u github --password-stdin
```

## 2. Docker stack

Download `docker-compose.yml`, `env.example`, and `NOTICE` from the **GitHub Release** you are deploying (replace `vX.Y.Z` below). Release candidates are marked **Pre-release** — take `docker-compose.yml` from that Pre-release’s assets. Default-branch compose is the latest **final** only — not RCs.

**bash / zsh / Git Bash / WSL:**

```bash
curl -fsSLO https://github.com/tracera-dev/tracera-releases/releases/download/vX.Y.Z/docker-compose.yml
curl -fsSLO https://github.com/tracera-dev/tracera-releases/releases/download/vX.Y.Z/env.example
curl -fsSLO https://github.com/tracera-dev/tracera-releases/releases/download/vX.Y.Z/NOTICE
cp env.example .env
```

**PowerShell:**

```powershell
$ver = 'vX.Y.Z'
$base = "https://github.com/tracera-dev/tracera-releases/releases/download/$ver"
Invoke-WebRequest -Uri "$base/docker-compose.yml" -OutFile docker-compose.yml
Invoke-WebRequest -Uri "$base/env.example" -OutFile env.example
Invoke-WebRequest -Uri "$base/NOTICE" -OutFile NOTICE
Copy-Item env.example .env
```

Fill every **Required** field in `.env` (see `env.example`). Empty Required values fail `docker compose up`. Booleans are `true` / `false` only.

Object storage is an internal S3 gateway (not on the host). Set `TRACERA_S3_SECRET_KEY` in `.env`; access key and bucket default to `tracera` (see `env.example`).

Production requires **SMTP** (`TRACERA_MAIL_TRANSPORT=smtp` in `.env`). Set `TRACERA_MAIL_SMTP_HOST`, `TRACERA_MAIL_FROM_ADDRESS`, and your SMTP credentials for the mail relay. Invite and password-reset links in emails are built from `TRACERA_PUBLIC_APP_URL` — set it to the same public HTTPS origin users open in the browser (for example `https://tracera.example.com:8088`) and set `TRACERA_COOKIE_SECURE=true`.

Do **not** use `TRACERA_MAIL_TRANSPORT=log` in production — the API would write invite and password-reset links (including one-time tokens) to its logs. Use `log` only for local troubleshooting.

Save the **signed license file** from Tracera support as `tracera.license` next to `docker-compose.yml` (default `TRACERA_LICENSE_HOST_PATH=./tracera.license`). Compose mounts that file into the API. If the file lives elsewhere, set `TRACERA_LICENSE_HOST_PATH` in `.env` (see `env.example`).

Then start the stack:

```bash
docker compose --env-file .env up -d
```

First boot: API runs migrations, bootstrap users, and S3 bucket setup automatically.

After the stack is up, open **Admin → License** to confirm status (the page also shows **Version** for support). To renew, replace the same file — click **Refresh** for an immediate update (or wait about a minute / restart the API container). Without a valid license the UI stays usable for **reading**; creating or changing data (including Autotest API writes) is blocked until a current license is installed.

### Health

**bash / zsh / Git Bash / WSL:**

```bash
curl -fsS http://127.0.0.1:8080/health
curl -fsS -o /dev/null -w '%{http_code}\n' http://127.0.0.1:8088/
```

**PowerShell:**

```powershell
Invoke-RestMethod http://127.0.0.1:8080/health
(Invoke-WebRequest http://127.0.0.1:8088/).StatusCode
```

Liveness returns `{ "status": "ok" }`. Ops diagnostics: `GET /health/detail` only when `TRACERA_ENABLE_HEALTH_DETAIL=true` (off by default → **404**). API docs: `GET /docs` on the API (and `{your-ui-url}/docs` via the UI proxy).

The UI sets CSP and related security headers — keep them if you add another reverse proxy. `/docs` uses a separate, relaxed CSP; do not replace it with the strict SPA policy at the edge.

## 3. Desktop client (optional)

Download installers for your OS from the same GitHub Release. Verify checksums against `SHA256SUMS` from the release:

**bash / zsh / Git Bash / WSL:**

```bash
sha256sum -c SHA256SUMS
```

**PowerShell** (per downloaded installer; compare to the matching line in `SHA256SUMS`):

```powershell
Get-FileHash .\Tracera_*.exe -Algorithm SHA256 | Format-List
```

On first launch, enter your **Tracera server URL** — the same address you open in the browser (for example `https://tracera.example.com:8088`). You can change it later under **User settings → Server**.

Optional IT override: set environment variable `TRACERA_DESKTOP_API_URL` to that origin before starting the app (locks the in-app Server field).

The web UI also offers **Open in Desktop** on the rail and auth screens; invite/reset emails stay normal `https://…` links.

**Desktop builds are unsigned** (no Apple notarization / Windows Authenticode):

| OS | First launch |
|----|----------------|
| **macOS** | Right-click the app → **Open** (Gatekeeper). Or allow via MDM. |
| **Windows** | SmartScreen → More info → **Run anyway**, or IT allowlist. |
| **Linux** | Usually installs and runs without extra steps. |
