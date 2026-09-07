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

Always take `docker-compose.yml` from the Release assets (final or Pre-release). Default-branch compose is the latest **final** only — not RCs.

Desktop installers are **unsigned** (no Apple / Windows code signing). See [INSTALL.md](INSTALL.md) for Gatekeeper / SmartScreen notes.

**License:** Tracera application software is proprietary. Copyright (c) 2026 Tracera. All rights reserved. Use requires a separate written agreement with Tracera. Open-source components included with or used by a deployment remain under their own licenses — see [NOTICE](NOTICE).

**Bugs and improvements:** open an issue with the [Bug](https://github.com/tracera-dev/tracera-releases/issues/new?template=bug.yml) or [Feature](https://github.com/tracera-dev/tracera-releases/issues/new?template=feature.yml) form (include your Tracera version from Admin → License; for adapters, include the adapter version too).
