---
project: dnd-merchants-generator
researched_at: 2026-06-15
recommended_platform: Render
runner_up: Railway
context_type: mvp
tech_stack:
  language: Python 3.14
  framework: Django 6.0
  runtime: CPython (WSGI / gunicorn)
---

## Recommendation

**Deploy on Render.**

Render is the only researched platform that offers a genuinely free ($0) tier that can actually
run a Django WSGI app, which matched the developer's stated cost priority. It scores Pass on all
five agent-friendly criteria (native CLI, managed runtime, markdown docs, scriptable deploy API,
GA MCP server) and supports the standard Django pattern (`build.sh` → `gunicorn wsgi`, WhiteNoise
for statics, `render.yaml` blueprint) with co-located managed Postgres — satisfying the "co-location
preferred" interview answer.

**Critical caveat, recorded up front:** the *free* tier cannot durably persist data. Free Postgres
self-deletes 30 days after creation and persistent disks (for SQLite) are paid-only. Because durable
persistence (FR-008) is the product's core insight, the realistic minimum viable configuration is the
**Starter plan (~$7/mo) plus a persistent disk for SQLite, or a paid Postgres tier with backups**.
Treat the free tier as a prototype/dev environment and budget ~$7–14/mo for the production instance.

## Platform Comparison

Hard filters were applied before scoring:

- **Netlify — dropped.** Functions support only JS/TS/Go and are short-lived; there is no way to host
  a Django WSGI app. Tech-stack runtime mismatch.
- **Vercel — kept, penalized.** Django now runs zero-config (wrapped as one Fluid-Compute function,
  Python 3.14 supported), but the filesystem is ephemeral/read-only — **SQLite is dead** and you must
  move to external Neon Postgres, plus accept stateless execution and cold starts. Poor fit for a
  SQLite-backed MVP with a hard persistence guarantee.
- **Cloudflare — kept, penalized.** Python Workers (Pyodide/WASM, beta) cannot host real Django (D1
  disables transactions, Admin partially broken). The only viable path is **Containers (GA 2026-04-13)**
  on the **paid** Workers plan ($5/mo) + external Postgres via Hyperdrive — the most complex option for
  a 3-week solo MVP, and notably **not free** despite the initial assumption.

Scoring of the platforms that can run Django (Pass / Partial / Fail):

| Platform | CLI-first | Managed/Serverless | Agent-readable docs | Stable deploy API | MCP / Integration | Notes |
|---|---|---|---|---|---|---|
| **Render** | Pass | Pass | Pass | Pass | Pass | Native CLI, GA MCP (Aug 2025), markdown docs, `render.yaml`. Only true $0 Django path. |
| **Railway** | Pass | Pass | Partial | Pass | Pass | Always-on by default (no spin-down), co-located Postgres w/ backups, MCP + Claude Code. ~$5/mo, no free always-on tier. |
| **Fly.io** | Pass | Pass | Pass | Pass | Fail | Mature `flyctl`, cheap SQLite-on-volume, `llms.txt` docs. No official MCP; managed Postgres floor ~$38/mo. |
| Vercel | Pass | Pass | Pass | Pass | Partial (beta) | SQLite impossible; forces Neon Postgres; stateless + cold starts. |
| Cloudflare | Pass | Partial (Containers) | Pass | Pass | Pass | Django only via paid Containers + external Postgres; highest complexity. |
| Netlify | — | — | — | — | — | Dropped: cannot host Django WSGI. |

### Shortlisted Platforms

#### 1. Render (Recommended)

Render wins on the developer's explicit cost driver: it is the only platform with a free tier that
runs a full Django process. Beyond that, its agent story is excellent — a real open-source native CLI
(`render deploys create --wait`, `render logs --tail`, `render psql`, `render ssh`), a GA MCP server
(20+ tools, works with Claude Code), markdown docs, and `render.yaml` infrastructure-as-code. Managed
Postgres, persistent disks, and Key Value are co-located in one region, matching the co-location
preference. The decisive weakness — recorded in the risk register — is that *free* and *durable* are
mutually exclusive here, so the durable production configuration costs ~$7–14/mo.

#### 2. Railway

The strongest runner-up and the better choice if the free-tier persistence trap proves unacceptable.
Railway is **always-on by default** (no spin-down, so no cold-start UX hit), offers one-click
**co-located Postgres with automated backups** (directly solving Render's free-Postgres deletion
risk), and has the best agent integration (official MCP server + documented Claude Code support).
The gap vs. Render: no free always-on tier (~$5/mo usage-metered minimum), the Railpack builder is
beta (March 2026), and Python 3.14 is not explicitly documented.

#### 3. Fly.io

The existing default in `tech-stack.md` (`deployment_target: fly`). Cheap SQLite-on-a-volume
(~$2–8/mo), full CPython, mature `flyctl`, and `llms.txt` docs make it a solid technical fit. It fell
to third because it scores **Fail** on the MCP/agent-integration criterion (no official MCP server)
and its *managed* Postgres floors at ~$38/mo — directly conflicting with the "minimize cost +
co-location preferred" interview answers. Strong fallback if you prefer SQLite-on-volume over a
managed DB and don't need MCP.

## Anti-Bias Cross-Check: Render

### Devil's Advocate — Weaknesses

1. **Free spin-down vs. the core UX promise.** Free web services spin down after 15 min idle and
   cold-start ~60s. The PRD's primary success criterion is "create a shop in <15s without breaking
   session tempo" — a 60s wait on the first request of a session blows that budget.
2. **$0 + durable persistence is impossible on Render.** Free Postgres self-deletes 30 days after
   creation (+14-day grace, no backups); persistent disks for SQLite are paid-only. A truly free
   deployment cannot durably store data — a direct violation of FR-008.
3. **Persistent disk blocks zero-downtime deploys and locks you to a single instance** once attached.
4. **Python 3.14 support unverified** — released late 2025; may not yet be a selectable Render runtime.
5. **No first-class CLI rollback** — rollback is dashboard/API only, weakening unattended agent recovery.

### Pre-Mortem — How This Could Fail

The team shipped on Render's free tier at $0 to hit the deadline, and it demoed perfectly. Then real
sessions started. Every time the DM opened the tool mid-session after a quiet stretch, the free
instance had spun down, and the table waited ~60 seconds for a cold start — the exact session tempo
the product promised to protect. Worse, they'd wired the free Postgres add-on; 30 days after creation
it expired, and with no backups on the free tier, a campaign's worth of saved shops vanished — an
unrecoverable breach of FR-008. The retro conclusion: "free" was never compatible with "durable." The
fix they'd resisted on cost grounds — the $7 Starter plan plus a persistent disk for SQLite (or paid
Postgres with backups) — was the actual minimum viable configuration all along. The free tier was fine
as a prototype and actively dangerous as the place real campaign data lived.

### Unknown Unknowns

- **Free Postgres deletion is silent and permanent** — 30 days after *creation*, not last use; no backups.
- **Free web's 750 instance-hours are per-workspace, not per-service** — a second free service competes
  for the same pool.
- **Cold starts compound with `migrate`** — a cold free instance running migrations on first hit adds
  further latency to that already-slow first request.
- **"Free" realistically becomes ~$7–14/mo the moment persistence matters** — plan it as a deliberate
  upgrade, not a post-data-loss fire drill.

## Operational Story

- **Preview deploys**: Render auto-deploys on push to the connected branch; PRs can spin up **Preview
  Environments** (defined in `render.yaml`) as ephemeral URLs. Free-tier previews inherit spin-down.
- **Secrets**: stored as **environment variables / environment groups** in the Render dashboard or via
  `render.yaml` (mark sensitive values `sync: false` so they aren't committed). `DJANGO_SECRET_KEY`,
  `DEBUG=False`, `ALLOWED_HOSTS`, and `DATABASE_URL` (if using Postgres) live here — not in the repo.
  Readable by workspace members; rotate by editing the var and redeploying.
- **Rollback**: redeploy a previous successful deploy from the dashboard (Deploys → "Rollback") or via
  the REST API. There is no first-class `render rollback` CLI subcommand. Note: DB migrations do **not**
  auto-roll-back — reverting code does not revert a schema change.
- **Approval**: agent may run `render deploys create --wait`, tail logs, and read status unattended.
  **Human-only**: creating/deleting the persistent disk, deleting or resizing the database, rotating
  `DJANGO_SECRET_KEY`, and any plan/billing change.
- **Logs**: `render logs --resources <service-id> --tail` (live), with time-range and `--output json`
  for structured parsing; or the GA MCP server (`https://mcp.render.com/mcp`) for typed tool access.

## Risk Register

| Risk | Source | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| Free Postgres self-deletes after 30 days → total data loss (violates FR-008) | Devil's advocate / Unknown unknowns | H | H | Do **not** store real data on free Postgres. Use SQLite on a **paid persistent disk**, or upgrade to a paid Postgres tier with backups, before any real campaign data is entered. |
| Free-tier spin-down cold start (~60s) breaks the <15s session-tempo success criterion | Devil's advocate / Pre-mortem | H | M | Use the **Starter plan (~$7/mo)** for the production instance (always-on). Keep free tier for dev/preview only. |
| SQLite data lost on redeploy if disk not mounted first | Research finding | M | H | Attach the persistent disk and confirm the mount path **before** writing any data; point Django's `DATABASES['default']['NAME']` at the disk path. |
| Persistent disk forces single instance + downtime on deploy | Research finding | M | L | Acceptable for solo single-user MVP. If multi-instance/zero-downtime later needed, migrate SQLite → managed Postgres. |
| Python 3.14 not yet a selectable Render runtime | Devil's advocate | M | M | Verify 3.14 availability (`.python-version` / `PYTHON_VERSION`); fall back to 3.13 if unsupported. Confirm before relying on 3.14-only features. |
| No first-class CLI rollback → agent can't auto-revert unattended | Devil's advocate | L | M | Script rollback via the REST API, or treat rollback as a human-gated dashboard action. |
| Realistic cost drifts to ~$7–14/mo despite "free" goal | Pre-mortem | H | L | Budget ~$7–14/mo now (Starter + disk, or Starter + paid Postgres) as a planned, not surprise, cost. |

## Getting Started

> These steps assume the repo is not yet on GitHub and dependencies are still unpinned
> (per CLAUDE.md). Pin dependencies first — Render builds from `requirements.txt`.

1. **Create `requirements.txt`** in a real `.venv` (don't pollute global Python):
   `py -m pip freeze > requirements.txt` after installing into a venv — at minimum `Django==6.0.6`,
   `gunicorn`, `whitenoise`, and (if using Postgres) `psycopg[binary]` + `dj-database-url`.
2. **Add a `build.sh`** (chmod +x): `pip install -r requirements.txt && python manage.py collectstatic --noinput && python manage.py migrate`.
3. **Configure Django for production**: move `SECRET_KEY` and `DEBUG` to env vars, add WhiteNoise to
   `MIDDLEWARE`, set `STATIC_ROOT`, and set `ALLOWED_HOSTS` from an env var (the scaffold ships
   `DEBUG=True` and a hardcoded key — fix before deploy).
4. **Pin the Python version**: add a `.python-version` file (try `3.14`; fall back to `3.13` if Render
   rejects it).
5. **Add `render.yaml`** at repo root: one `web` service with `buildCommand: ./build.sh`,
   `startCommand: gunicorn dnd_merchants_generator.wsgi:application`, a **persistent disk** mounted for
   the SQLite file, and `DJANGO_SECRET_KEY` (`generateValue: true`), `DEBUG=False`, `ALLOWED_HOSTS`.
6. **Deploy**: connect the GitHub repo in the Render dashboard (Blueprint), or
   `render deploys create <service-id> --wait` once the service exists. Tail with
   `render logs --resources <service-id> --tail`.

## Out of Scope

The following were not evaluated in this research:
- Docker image configuration / Dockerfiles
- CI/CD pipeline setup (GitHub Actions auto-deploy is noted in `tech-stack.md` but not configured here)
- Production-scale architecture (multi-region, HA, disaster recovery)
