# Repository Guidelines

Onboarding for AI agents working in **dnd-merchants-generator** — a Django 6.0 web app for
D&D 5e Dungeon Masters to generate, persist, and price-adjust merchant shops. Single
user-per-data MVP, SQLite, deploy target Fly.io. Assumes you know Django; documents only
what is specific to this repo. `@CLAUDE.md` is the condensed auto-loaded summary of this
file; `@context/foundation/prd.md` (Polish) is the authoritative requirements spec.

## Critical, read first

- **Run management commands with `py`, not `python` or `uv`.** `context/foundation/tech-stack.md`
  records `package_manager: uv`, but **uv is not installed and no virtualenv exists**. Django
  6.0.6 was installed into global Python 3.14 via `py -m pip`. Use `py manage.py <cmd>`.
- **Dependencies are unpinned** — there is no `requirements.txt`, `pyproject.toml`, or lockfile.
  Before adding any dependency, create a real `.venv` first so you don't further pollute the
  global interpreter.
- **Per-user data isolation is a hard requirement, not a nicety.** Every shop/merchant query
  MUST be scoped to `request.user`. No DM may ever see another DM's data; there is no
  admin/shared role in the MVP. Treat a queryset that isn't owner-filtered as a bug.
- **The Python package is `dnd_merchants_generator` (underscores).** The product name is
  `dnd-merchants-generator` (hyphens), which isn't a valid identifier. Settings module:
  `dnd_merchants_generator.settings`.

## Setup & commands

```sh
# One-time isolated env (currently MISSING — deps live in the global interpreter)
py -m venv .venv
.venv\Scripts\activate                 # PowerShell/cmd on Windows
py -m pip install django               # then pin into requirements.txt / pyproject.toml

# Day-to-day (works against the global env today)
py manage.py migrate                                   # apply migrations (run before first start)
py manage.py runserver                                 # dev server at http://127.0.0.1:8000/
py manage.py createsuperuser                           # exercise built-in auth/admin
py manage.py startapp <name>                           # new domain app — then add to INSTALLED_APPS
py manage.py makemigrations <app>                      # after model changes
py manage.py test                                      # full suite (none exist yet)
py manage.py test <app>.<TestClass>.<test_method>      # single test
```

## Structure

The repo is bare `startproject` output — **no domain app exists yet**.

- `@manage.py`, `dnd_merchants_generator/` — Django project config (`settings.py`, `urls.py`,
  `wsgi.py`/`asgi.py`). `INSTALLED_APPS` currently holds only `contrib.*` defaults.
- New domain code goes in **app packages you create** with `startapp` (e.g. a `shops` app for
  the shop/merchant/item models and the two business rules below). Register each new app in
  `INSTALLED_APPS` in `dnd_merchants_generator/settings.py`.
- `context/` — agent-context foundation, **not** application code:
  - `@context/foundation/prd.md` — canonical product spec (in **Polish**). Read before building
    any feature.
  - `@context/foundation/tech-stack.md` — stack rationale and feature flags.
  - `context/changes/` — per-change working dirs. **Never write to `context/archive/`**
    (immutable; open a new change instead).

## Domain rules (the core of the product)

Two rules carry the product; everything else is standard CRUD + Django auth. See the PRD
Business Logic section for authoritative detail.

1. **Assortment generation** — draw a random item set from a built-in SRD-derived catalog,
   filtered so item type and power/value match the shop's assortment type and town wealth.
   Randomness happens *within* a matched pool, not across the whole catalog (target ≥ 90% fit).
2. **Persuasion price recompute** — one Persuasion roll recomputes the displayed prices of the
   **entire** item list against discount thresholds in < 1 s. **Base prices must be preserved** —
   this is a display transform, never a destructive write. Default thresholds (tunable, see PRD
   Open Questions): <10 → 0%; 10–14 → −5%; 15–19 → −10%; 20+ → −20%.

Persistence guarantee (`FR-008`): reopening a saved shop shows the exact same items and base
prices as when it was saved.

## Scope guardrails (PRD Non-Goals — don't build these)

No sharing/public catalog · no inventory/stock tracking (buying doesn't decrement stock) ·
no player-facing view (DM eyes only) · single currency only (gold pieces) · no
custom-artifact creator (built-in SRD data only).

## Conventions

- **Currency is gold pieces only** — store prices as integers in gold; do not model
  copper/silver/electrum/platinum conversions.
- `SECRET_KEY` is hardcoded and `DEBUG = True` in `settings.py` (scaffold defaults) — move both
  to environment-driven config before any deploy. Deploy target is Fly.io; CI is GitHub Actions
  with auto-deploy on merge (per `tech-stack.md`).
- Not yet a git repository; no commit conventions established.
