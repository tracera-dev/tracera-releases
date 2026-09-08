# Tracera releases

Public distribution channel for Tracera on-prem **artifacts only** — no application source code.

**Product site:** [tracera.dev](https://tracera.dev) · **Contact:** [contact@tracera.dev](mailto:contact@tracera.dev)

| Resource | Location |
|----------|----------|
| Install guide | [INSTALL.md](INSTALL.md) |
| Stack compose (digest-pinned) | [GitHub Releases](https://github.com/tracera-dev/tracera-releases/releases) — download `docker-compose.yml` from the release you deploy |
| Env template | [env.example](env.example) |
| Third-party notices | [NOTICE](NOTICE) |
| Desktop installers | [GitHub Releases](https://github.com/tracera-dev/tracera-releases/releases) |
| Container images | Private — `docker login` with the token from Tracera support (see [INSTALL.md](INSTALL.md)) |

**License:** Tracera application software is proprietary. Copyright (c) 2026 Tracera. All rights reserved. Use requires a separate written agreement with Tracera. Open-source components included with or used by a deployment remain under their own licenses — see [NOTICE](NOTICE).

Download `docker-compose.yml` from the GitHub Release you are deploying. After a **final** release, that same file is also on the default branch. For a **pre-release (RC)**, use that Pre-release’s compose asset — do not use default-branch compose for an RC.

Desktop installers are **unsigned** (no Apple / Windows code signing). See INSTALL.md for Gatekeeper / SmartScreen notes.
