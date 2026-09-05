# Tracera on-prem install

Artifacts and release notes: [GitHub Releases](https://github.com/tracera-dev/tracera-releases/releases).

## Requirements

- Docker Engine + Compose v2 (BuildKit not required for pull-only deploy)
- ≥ **8 GB RAM** all-in-one (medium profile)
- Outbound HTTPS to `ghcr.io` for image pull

## 1. GHCR login (private images)

Tracera support provides a **fine-grained GitHub PAT** with **read:packages** (one token per customer).

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

Download `docker-compose.yml`, `env.example`, and `NOTICE` from the **GitHub Release** you are deploying (replace `vX.Y.Z` below).

**bash / zsh / Git Bash / WSL:**

```bash
curl -fsSLO https://github.com/tracera-dev/tracera-releases/releases/download/vX.Y.Z/docker-compose.yml
curl -fsSLO https://github.com/tracera-dev/tracera-releases/releases/download/vX.Y.Z/env.example
curl -fsSLO https://github.com/tracera-dev/tracera-releases/releases/download/vX.Y.Z/NOTICE
cp env.example .env
# edit .env — passwords, TRACERA_PUBLIC_APP_URL, TRACERA_SECRETS_ENCRYPTION_KEY, SMTP, …
# Compose loads `.env` into the API container (mail and other settings). Keep
# TRACERA_DATABASE_URL / TRACERA_S3_ENDPOINT out of .env — compose sets those.
docker compose --env-file .env up -d
```

**PowerShell:**

```powershell
$ver = 'vX.Y.Z'
$base = "https://github.com/tracera-dev/tracera-releases/releases/download/$ver"
Invoke-WebRequest -Uri "$base/docker-compose.yml" -OutFile docker-compose.yml
Invoke-WebRequest -Uri "$base/env.example" -OutFile env.example
Invoke-WebRequest -Uri "$base/NOTICE" -OutFile NOTICE
Copy-Item env.example .env
# edit .env — passwords, TRACERA_PUBLIC_APP_URL, TRACERA_SECRETS_ENCRYPTION_KEY, SMTP, …
# Compose loads `.env` into the API container (mail and other settings). Keep
# TRACERA_DATABASE_URL / TRACERA_S3_ENDPOINT out of .env — compose sets those.
docker compose --env-file .env up -d
```

Images are pinned by **digest** in `docker-compose.yml`. Semver tags are listed in release notes for reference.

First boot: API runs migrations, bootstrap users, and S3 bucket setup automatically.

Object storage uses an internal SeaweedFS S3 gateway (not published on the host). Set `TRACERA_S3_ACCESS_KEY`, `TRACERA_S3_SECRET_KEY`, and `TRACERA_S3_BUCKET` in `.env` (see `env.example`). There is no object-storage admin console in the stack.

### Email (invite / password reset)

Production requires **SMTP** (`TRACERA_MAIL_TRANSPORT=smtp` in `.env`). Set `TRACERA_MAIL_SMTP_HOST`, `TRACERA_MAIL_FROM_ADDRESS`, and your SMTP credentials for the mail relay. Invite and password-reset links in emails are built from `TRACERA_PUBLIC_APP_URL` — set it to the same public HTTPS origin users open in the browser (for example `https://tracera.example.com:8088`).

Do **not** use `TRACERA_MAIL_TRANSPORT=log` in production — the API would write invite and password-reset links (including one-time tokens) to its logs. Use `log` only for local troubleshooting.

Password reset recovers a forgotten **password** only. It does **not** turn off two-factor authentication (2FA).

### Two-factor authentication (lost authenticator)

When a user enables 2FA, Tracera shows **one-time backup codes**. Keep them somewhere safe — each code works once and can unlock sign-in or turn 2FA off if the authenticator app is lost.

If backup codes are also unavailable, an **admin** can open **Admin → Users**, choose the user, and use **Disable 2FA**. The user then signs in with password only and can enable 2FA again.

Health:

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

Liveness returns `{ "status": "ok" }`. Detailed diagnostics (`tenantBootstrapped`, database pool) are available at `GET /health/detail` only when `TRACERA_ENABLE_HEALTH_DETAIL=true` in `.env` (disabled by default in production; when disabled the endpoint is not registered and returns **404**). Boolean settings in `.env` use `true` or `false` only (not `1` / `0`). API docs UI is always at `GET /docs` on the API (and `{your-ui-url}/docs` through the UI reverse-proxy).

The UI container sets browser security headers (`Content-Security-Policy`, `X-Frame-Options: DENY`, `Referrer-Policy: no-referrer`, and related). If you put another reverse proxy in front, leave those response headers intact. API docs at `{your-ui-url}/docs` uses a **separate, relaxed** CSP (bundled Scalar script + fonts); do not override it with the strict SPA policy at the edge.

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

From the web UI you can also use **Open in Desktop** (icon rail or sign-in / invite / reset screens). Invite and password-reset email links stay normal `https://…` addresses; if the desktop app is installed, those pages offer to open in the app.

**Desktop builds are unsigned** (no Apple notarization / Windows Authenticode). That is intentional for this distribution channel:

| OS | First launch |
|----|----------------|
| **macOS** | Right-click the app → **Open** (Gatekeeper). Or allow via MDM. |
| **Windows** | SmartScreen → More info → **Run anyway**, or IT allowlist. |
| **Linux** | Usually installs and runs without extra steps. |

## Pre-releases (RC)

Release candidates are marked **Pre-release**. Use their compose asset for testing; do not assume the default-branch compose tracks an RC.

## License

Tracera application software is proprietary. Copyright (c) 2026 Tracera. All rights reserved. Use requires a separate written agreement with Tracera. Third-party open-source components remain under their own licenses — see [NOTICE](NOTICE).

## Support

Treat the GHCR token as a secret; rotate it after a leak or when access should end. Do not commit `.env` or your GHCR token to git.
