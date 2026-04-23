# Documentation Pipeline Setup

## Context

ExpertFlow CX Documentation uses a Documentation as Code (DaC) approach.
Docs are written in Markdown, tracked in Git, reviewed through automated
pipelines, and published automatically to a Docusaurus static site on merge
to `main`.

This document tracks the setup state of all pipelines and what remains to be done.

---

## Architecture: 6-Step Setup Plan

| Step | What | Status |
|------|------|--------|
| Step 1 | GitHub repository created | ✅ Done |
| Step 2 | Docusaurus installed | ✅ Done |
| Step 3 | Pipeline 3 — Production publish live | ✅ Done |
| Step 4 | Pipeline 1 — PR validation gate | ❌ Pending |
| Step 5 | Pipeline 2 — Preview deploys | ❌ Pending |
| Step 6 | Pipeline 4 — Scheduled maintenance | ❌ Pending |

---

## Current State

| What | Detail |
|------|--------|
| GitHub repo | `expertflow/expertflow-cx-documentation` (public) |
| Live docs site | https://expertflow.github.io/expertflow-cx-documentation/ |
| Docusaurus location | `docs-site/` folder |
| Production publish workflow | `.github/workflows/deploy-docs.yml` |
| Triggers on | Push to `main` touching `docs-site/**`, `Phase4/**`, `Restructured/**` |
| PR validation | Not set up |
| Preview deploys | Not set up |
| Scheduled maintenance | Not set up (`check-links.js` exists but not wired to a schedule) |

---

## What Is Already Done (No Action Needed)

### Step 1 — GitHub Repository
Repo exists at `expertflow/expertflow-cx-documentation`, public, default branch `main`.

### Step 2 — Docusaurus Installed
Docusaurus is installed inside `docs-site/`. Config files `docusaurus.config.js`
and `sidebars.js` are in place.

### Step 3 — Pipeline 3: Production Publish
File: `.github/workflows/deploy-docs.yml`

Triggers automatically when anything in `docs-site/**` is pushed to `main`.
Builds Docusaurus and deploys to GitHub Pages.

```
Push to main → GitHub Actions builds docs-site → Deploys to GitHub Pages → Site updates
```

---

## What Needs to Be Done

### Step 4 — Pipeline 1: PR Validation Gate

**What it does:** Runs automatically on every Pull Request that touches
`docs-site/docs/**`. Blocks the PR if any check fails.

**Checks (in order):**
1. Markdownlint — heading hierarchy, list formatting
2. Spell check — `cspell` with custom dictionary for product terms
3. No placeholder text — flags `TODO`, `FIXME`, `TBD` in docs
4. Docusaurus build — if build breaks, PR is blocked

**How to implement:**
- Create `.github/workflows/validate-docs.yml` on a feature branch
- Open a PR to test it before merging

**Prerequisite:** Enable branch protection on `main` in GitHub repo settings
(require PR + passing status checks before merge). Without this, the gate
can be bypassed.

**Effort:** ~2 hours

---

### Step 5 — Pipeline 2: Preview Deploys

**What it does:** Every PR gets a unique temporary live URL so reviewers see
the rendered Docusaurus site — not raw Markdown — before approving.

**Recommended tool: Vercel (free tier)**
- Connect the GitHub repo to Vercel
- Vercel auto-creates a preview URL per PR (e.g. `expertflow-cx-docs-pr-42.vercel.app`)
- Posts the URL as a comment on the PR — no extra config needed
- Production site on GitHub Pages is unaffected

**Alternative:** Netlify — identical capability.

**Effort:** ~10 minutes to connect Vercel. No new GitHub Actions workflow needed.

---

### Step 6 — Pipeline 4: Scheduled Maintenance

**What it does:** A weekly automated job that checks for rot and opens a
GitHub Issue if problems are found.

**Checks:**
1. External link checker — runs against the built site
2. Stale content report — flags pages with `last_updated` older than 60 days

**Note:** `check-links.js` already exists in the repo and can be reused here.

**Trigger:** GitHub Actions cron — every Monday 08:00

**How to implement:**
- Create `.github/workflows/scheduled-maintenance.yml`
- Wire `check-links.js` into it
- Add stale content scan step
- On failure: auto-open a GitHub Issue titled `[Docs Maintenance] Weekly check failed — YYYY-MM-DD`

**Effort:** ~1 hour

---

## Implementation Order (Recommended)

```
Week 1
  ├── Enable branch protection on main (5 min, GitHub settings)
  ├── Step 4 — Pipeline 1: PR validation  (~2 hrs)
  └── Step 5 — Pipeline 2: Connect Vercel (~10 min)

Week 2
  └── Step 6 — Pipeline 4: Scheduled maintenance (~1 hr)
```

---

## Why This Matters for a Collaborative Team

Without these gates anyone can push broken or unreviewed docs directly to
the live site. These pipelines ensure:

- Every doc is reviewed before publishing
- The site never breaks from a bad push
- Dead links and stale content are caught automatically
- Reviewers see the real rendered output, not raw Markdown diffs

---

## Cost Summary

| Item | Cost |
|------|------|
| GitHub Actions (all workflows) | Free (public repo) |
| Vercel preview deploys | Free tier |
| Branch protection | Free (GitHub feature) |
| All tooling (markdownlint, cspell, lychee) | Free, open source |

**Total additional cost: $0**
