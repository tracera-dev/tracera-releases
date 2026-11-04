# Tracera high availability

Default install ([INSTALL.md](INSTALL.md)) runs **one** API container that serves HTTP and background work (`TRACERA_ROLE=all`). That is enough for most on-prem sites.

Use this playbook when you need **more than one** API process (Kubernetes, several VMs, or a load balancer in front of the API).

## What to scale

| Layer | Scale | Notes |
|-------|--------|--------|
| **UI** | Stateless replicas OK | Reverse-proxy to the API service |
| **API** (`TRACERA_ROLE=api`) | Horizontal replicas OK | Put a load balancer / Service in front; sticky sessions are **not** required (session cookie is validated in Postgres) |
| **Worker** (`TRACERA_ROLE=worker`) | 1+ replicas | Runs jobs, mail outbox, maintenance, blob cleanup. Exposes `/health` only |
| **PostgreSQL** | **One** logical database | Operator HA (primary + standby) is fine; do **not** point Tracera at several independent Postgres primaries |
| **Object storage (S3)** | **One** logical bucket/endpoint | Same rule — one store Tracera talks to |

Do **not** treat Postgres or S3 like the API (N equal writable app pods). Tracera assumes one shared database and one shared object store.

## Process roles

Set `TRACERA_ROLE` on each API image container:

| Value | HTTP | Background workers |
|-------|------|--------------------|
| `all` (default) | Full API | Yes — single-container install |
| `api` | Full API | No — scale these behind the balancer |
| `worker` | `/health` only | Yes — job / mail / maintenance |

When you split roles, run **at least one** `worker` (or jobs and mail will stall). Keep the same `.env` secrets, database URL, S3 endpoint, and license mount on every replica.

Optional: set `TRACERA_DB_APPLICATION_NAME` to `tracera-api` / `tracera-worker` so Postgres `pg_stat_activity` shows which role connected.

## How Tracera stays consistent across replicas

- **Jobs** use leases (`worker_id` + expiry). Another worker reclaims only after the lease expires.
- **Schema migrate** takes a Postgres advisory lock so several pods can start safely.
- **Tree / run / analytics caches** are process-local LRUs. Mutating pods publish invalidation through Postgres (`NOTIFY`); every pod listens and clears its own cache. After a listen disconnect, that pod flushes its local caches.

You do **not** need Redis or an external cache bus for this.

## Suggested topologies

**Single host (compose):** keep `TRACERA_ROLE` unset / `all` — [INSTALL.md](INSTALL.md).

**Split API + worker (compose / K8s):**

1. Deploy Postgres + S3 (or managed equivalents) once.
2. Run N× `TRACERA_ROLE=api` behind the UI / ingress.
3. Run ≥1× `TRACERA_ROLE=worker` (same image, same env except role).
4. Health-check all pods on `GET /health`.
5. Point the UI upstream at the **api** Service only (not worker).

**RAM:** still plan from about **4 GB** for a small instance ([INSTALL.md](INSTALL.md)). Extra API replicas need their own memory on top of Postgres and S3.

## Out of scope

- Multi-primary Postgres or multi-writer S3 clusters as a Tracera feature
- Changing customer digest `docker-compose.yml` to multi-API by default (single `api` with `all` remains the install path)

Questions: [contact@tracera.dev](mailto:contact@tracera.dev).
