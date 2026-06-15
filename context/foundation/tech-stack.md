---
starter_id: django
package_manager: uv
project_name: dnd-merchants-generator
hints:
  language_family: python
  team_size: solo
  deployment_target: fly
  ci_provider: github-actions
  ci_default_flow: auto-deploy-on-merge
  bootstrapper_confidence: verified
  path_taken: standard
  quality_override: false
  self_check_answers: null
  has_auth: true
  has_payments: false
  has_realtime: false
  has_ai: false
  has_background_jobs: false
---

## Why this stack

A solo Mistrz Gry building a DnD merchants generator as a small web app on a hard
3-week deadline needs the boring-but-required half — email/password auth, per-user
data isolation, durable persistence, and CRUD over saved shops — to come for free.
Django is the recommended default for `(web, python)` and ships exactly that:
ORM, migrations, auth, and admin on day 1, leaving the calendar for the two real
domain rules (matched-pool assortment draw and the Persuasion price recompute),
which are plain Python on top. It clears three of four agent-friendly gates
outright; the typed gate is the Python-family caveat (Django is duck-typed, not
statically typed), but its convention discipline, large training corpus, and
current docs more than compensate, and bootstrapper confidence is verified, so
scaffolding will be smooth. Auth and persistence feature flags are set; payments,
realtime, AI, and background jobs are out of scope per the PRD non-goals.
Deployment targets Fly.io (the Django card default) with GitHub Actions running
auto-deploy on merge — the standard shape for a solo project.
