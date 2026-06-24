# DaC Pipeline × EFSpecCatalog: Discovery & Integration Analysis

**Session Date:** 2026-06-24
**Analyst:** Mary — Strategic Business Analyst (BMAD)
**Prepared for:** Navira, DaC Initiative Lead, ExpertFlow
**Status:** Draft — Pending Team Review and Prioritisation

---

## Purpose of This Document

This document captures the findings of a structured discovery session conducted to evaluate
how the ExpertFlow **Documentation as Code (DaC) Pipeline** initiative can be re-grounded
to use **EFSpecCatalog** as its authoritative source of truth for product specifications.

The session reviewed:
- The full DaC Pipeline design (problem statement, product brief, architecture, open decisions)
- The EFSpecCatalog structure, schema, content, and gap coverage data
- The strategic relationship between the two initiatives
- Six concrete integration opportunities with value assessment
- A set of open decisions required before any implementation can proceed

This document is intended for manager and team review. The Action Items section at the end
is the primary agenda for the follow-up discussion.

---

## Background: Two Initiatives, One Strategic Opportunity

### The DaC Pipeline (What It Is Today)

ExpertFlow's CX development teams produce documentation as a release-gate ritual — written
after features ship, under deadline pressure, from memory, in Confluence. The DaC Pipeline
was designed to fix this by embedding documentation into the GitHub PR workflow:

1. Developer opens a feature PR — no extra steps required
2. **AI bot** (GitHub Action + Claude API) analyzes the PR title, description, and code diff
   and generates a Markdown doc draft committed to the same branch
3. **Developer reviews** the draft — not writes it. A GitHub status check signals yellow
   (unreviewed) or green (reviewed), but never blocks merge
4. **Feature merges** → doc auto-publishes to Docusaurus in `draft` state (internal only)
5. **Pipeline creates a GitHub Issue** assigned to the Product Owner with a link to the draft
6. **PO reviews asynchronously**, edits if needed, closes the Issue
7. **Pipeline flips the doc** from `draft` to `published` — customer-visible on Docusaurus

The entire system runs as GitHub Actions. No servers, no database, no external CI tools.

**Current source for doc generation:** PR title + PR description + code diff only.

### EFSpecCatalog (What It Is)

EFSpecCatalog is the ExpertFlow **Agentic Specification Catalog** — the operational source of
truth for ExpertFlow's product capabilities, competitive positioning, and differentiation
principles. It drives CPQ qualification, competitive sales briefs, presales offer assembly,
public LLM discovery, and engineering commitment gates.

| Layer | Contents | Count |
|---|---|---|
| `features/` | Atomic capability specs (YAML + Markdown) | 84+ features, 7 domains |
| `solution-patterns/` | Customer-facing capability bundles | 5 patterns |
| `canon/axioms/` | Differentiation principles with competitive evidence | 18 axioms |
| `requirements/` | Customer asks, RFP signals, market demands | 3+ requirements |

Each feature spec carries rich structured metadata:

| Field | Purpose |
|---|---|
| `id` | Stable machine reference (e.g., `routing-00001`) |
| `name` | Human-readable feature name |
| `status` | Lifecycle: `backlog → under_implementation → live` |
| `domain` | One of 7 product domains |
| `public_description` | Customer-facing feature description |
| `canonical_use_case` | Specific customer scenario where this feature wins |
| `canon_axiom_refs` | Links to differentiating principles this feature expresses |
| `solution_pattern_refs` | Which solution patterns include this feature |
| `depends_on` | Prerequisite feature IDs (AND/OR gates) |
| `effort_estimate` | Engineering days |
| `provenance` | Authors, Jira tickets, funding source |

Features have a governance-gated lifecycle: the `backlog → under_implementation` transition
requires Top Management approval enforced via GitHub CODEOWNERS.

---

## Key Finding 1: The Confluence Gap Is Much Larger Than the Pipeline Assumes

The EFSpecCatalog Gap Monitor (run 2026-06-01) reveals the scale of what needs to move:

| Content Type | Confluence Pages | Pages to Migrate | Currently Covered | % Done |
|---|---:|---:|---:|---:|
| Features | 284 | 269 | 1 | **0.4%** |
| User Guides | 634 | 634 | 4 | **0.6%** |
| Solution Patterns | 485 | 485 | 4 | **0.8%** |
| Architecture | 66 | 41 | 0 | **0.0%** |
| API Reference | 64 | 60 | 2 | **3.3%** |
| Compliance | 214 | 177 | 4 | **2.3%** |
| Unmapped | 562 | 192 | 0 | **0.0%** |
| **TOTAL** | **2,309** | **1,858** | **15** | — |

**Implication for DaC:** The current pipeline design handles *new* docs from *new* PRs. It has
no mechanism for the 1,843 pages of existing Confluence content that needs to migrate. These
are not future docs — they are existing product knowledge that customers and support agents
are relying on today. The DaC initiative is incomplete without a migration strategy.

---

## Key Finding 2: The Current Pipeline Is Flying Blind Relative to the Product Spec

The AI bot currently generates docs from the code diff. This means it knows:

- **What code changed** — accurate
- **How the developer described the change** (PR description) — variable quality
- **Who authored the PR** — available

It does not know:

- **Which EFSpecCatalog feature this PR relates to** — not connected
- **The authoritative customer-facing description of the feature** — not available to the bot
- **The canonical use case** — not available
- **Which Canon Axioms this feature expresses** — not available
- **Whether the generated language is competitively grounded** — no check exists

A doc generated from a code diff describes *what changed in the code*.
A doc grounded in a feature spec describes *what the product does and why customers care*.

These are structurally different outputs. The first is technically accurate. The second is
product-accurate, customer-language-consistent, and competitively grounded.

---

## Key Finding 3: EFSpecCatalog Is Already Structured for This Integration

The EFSpecCatalog feature schema has fields that map directly onto DaC pipeline needs:

| EFSpecCatalog field | DaC Pipeline use |
|---|---|
| `public_description` | Primary input for AI doc body generation |
| `canonical_use_case` | Sets customer-voice tone and scenario framing |
| `canon_axiom_refs` | Injects competitive differentiation language |
| `solution_pattern_refs` | Provides "where this fits" context in the doc |
| `status` | Can trigger pipeline events (stub creation, publish gate) |
| `id` | Becomes `feature_id` in published doc frontmatter |
| `depends_on` | Could inform "related features" section in docs |
| `effort_estimate` / `commitment_ref` | Governs which features have docs in scope |

No schema changes are needed in EFSpecCatalog to enable this integration. The data is
already there. The integration gap is on the pipeline side.

---

## The Six Integration Opportunities

### Opportunity 1: Spec-Grounded Doc Generation *(Highest Value)*

**Current:** `PR diff → AI bot → doc draft`

**Upgraded:** `PR diff + EFSpecCatalog feature spec → AI bot → doc draft`

When a PR is tagged with a feature ID (e.g., `routing-00001`), the pipeline fetches:
- `public_description`
- `canonical_use_case`
- Axiom claims from `canon_axiom_refs`
- Solution pattern membership from `solution_pattern_refs`

The AI generates a doc that is simultaneously accurate to the code change AND consistent
with the authoritative product specification. The developer review step validates the
technical accuracy. The PO review step validates the product accuracy against the spec.

**Value:** Eliminates the biggest risk in AI-generated docs — technically correct but
strategically incoherent content reaching customers.

---

### Opportunity 2: Canon Axiom Language in Every Customer-Facing Doc *(Differentiation Value)*

Each Canon Axiom carries a carefully crafted one-sentence competitive claim, authored by
the SA team to be precise, defensible, and customer-relevant. For example:

> *"ExpertFlow routes contacts by scoring agents on real-time skills and live state at the
> moment of dispatch — unlike queue-membership models that poll state on a fixed cycle —
> which means multi-skill agents are used optimally and assignment lag is eliminated in
> high-volume contact centres."*

When the AI bot knows which axioms apply to the feature being documented (via
`canon_axiom_refs`), it can use this language as a guiding principle for the customer-
facing narrative — ensuring every published doc reinforces ExpertFlow's competitive
positioning without requiring manual copy-editing.

**Value:** Competitive differentiation language propagates from EFSpecCatalog into all
customer documentation automatically. No manual step required.

---

### Opportunity 3: Feature ID Metadata in All Published Docs *(Traceability Value)*

Every published Docusaurus doc gets `feature_id: routing-00001` in its frontmatter.

This creates a **bidirectional link**:
- EFSpecCatalog feature → published doc URL
- Published doc → feature spec (for readers who want the spec)

Coverage tracking shifts from *"how many PRs have docs"* to *"how many catalog features
have live published docs"* — which is the correct unit of measurement for a product team
and maps directly to the gap monitor scorecard.

**Value:** Documentation completeness becomes auditable at the feature level. The gap monitor
becomes the documentation coverage dashboard.

---

### Opportunity 4: Status-Triggered Pipeline Events *(Automation Value)*

EFSpecCatalog feature lifecycle transitions happen via CODEOWNERS-gated PRs — GitHub Actions
can listen to them.

| EFSpecCatalog event | DaC Pipeline action |
|---|---|
| Feature → `under_implementation` | Create doc stub in Docusaurus (internal, `draft` state) |
| Feature → `live` | Trigger doc completeness check; assign PO review if doc is absent |
| Feature → `deprecated` | Flag published doc for archiving; notify PO |

This replaces "one PR = one doc trigger" with "one feature = one doc lifecycle" — the
pipeline becomes aware of the product roadmap, not just individual commits.

**Value:** Docs are created proactively when features enter implementation, not reactively
after they ship. No feature goes live without a doc in flight.

---

### Opportunity 5: Spec-Anchored PO Review *(Quality Gate Value)*

Currently, the PO review step creates a GitHub Issue with a link to the draft doc. The PO
reviews it cold — no context about what the spec says the feature should do.

**Upgraded:** The GitHub Issue includes, alongside the doc link:
- The `public_description` from EFSpecCatalog as the acceptance criterion
- The `canonical_use_case` as the customer-scenario check
- The axiom claim (if applicable) as the language quality standard

The PO is no longer reviewing a doc in isolation. They are reviewing whether the doc
faithfully represents the product spec. The spec provides the pass/fail standard.

**Value:** PO review becomes faster, more focused, and produces more consistent quality
outcomes. Reduces the number of review cycles before `published` state is reached.

---

### Opportunity 6: Confluence Migration via the DaC Pipeline *(Migration Value)*

The gap monitor establishes that 1,843 pages of Confluence content need structured migration.
The DaC pipeline, upgraded with EFSpecCatalog integration, becomes the migration vehicle:

| Phase | Mechanism | Output |
|---|---|---|
| **Map** | `/bmad-backlog-classifier` in EFSpecCatalog maps Confluence pages to feature IDs | Mapping table: Confluence page → feature ID |
| **Intake** | `/bmad-catalog-intake` generates proper feature specs from Confluence page content | Feature spec files in EFSpecCatalog |
| **Seed** | DaC pipeline generates initial docs from spec + Confluence content as context seed | Draft docs in Docusaurus |
| **Review** | PO review closes the loop using spec as acceptance criteria | Published docs |
| **Decommission** | Confluence page archived once doc is `published` | Migration scorecard updated |

This transforms Confluence migration from a manual archaeology project into a structured,
pipeline-driven flow. The gap monitor scorecard becomes the migration progress tracker.

**Value:** The DaC pipeline handles both new features (PRs) and legacy features (Confluence
migration) through the same structured, spec-grounded process.

---

## The Upgraded Pipeline: Before and After

| Dimension | Current DaC Pipeline | Upgraded DaC + EFSpecCatalog |
|---|---|---|
| **Doc source** | PR diff + developer description | PR diff + EFSpecCatalog feature spec |
| **Doc unit** | One PR = one doc | One feature = one doc lifecycle |
| **Language quality** | Developer-generated | Spec-grounded + axiom-consistent |
| **Coverage tracking** | By PR count | By feature ID against catalog |
| **Confluence migration** | Out of scope | Pipeline extension via catalog mapping |
| **PO review context** | Draft doc only | Draft doc + authoritative spec |
| **Pipeline triggers** | PR opened / PR merged | PR events + feature status transitions |
| **Traceability** | PR → doc | Feature spec ↔ doc ↔ PR |

**In one sentence:** The DaC pipeline stops being a documentation automation tool and
becomes the **publication layer of EFSpecCatalog**.

---

## Action Items and Next Steps

*The following questions are the agenda for the team and manager review session.
They are sequenced: earlier decisions unblock later ones.*

---

### Decision 1 — The Lynchpin: How Does a PR Declare Its Feature ID? `[MUST DECIDE FIRST]`

Everything else in this integration depends on answering: *how does the pipeline know
which EFSpecCatalog feature a PR relates to?*

**Options:**

| Option | How it works | Effort | Risk |
|---|---|---|---|
| A — PR template field | Developer fills `feature_id: routing-00001` in the PR template | Low | Discipline dependency — developers must remember |
| B — PR label | Developer applies GitHub label `feature: routing-00001` | Low | Same discipline dependency, lower friction |
| C — AI inference | Bot infers feature ID from PR diff using EFSpecCatalog as context | Medium | Some error rate; needs human confirmation flow |
| D — Hybrid (recommended) | Bot infers, flags uncertainty, developer confirms inline | Medium | Best balance of automation and accuracy |

**Questions for team:**
- Can we mandate a PR template field as part of DoD without creating friction?
- Is the EFSpecCatalog feature taxonomy stable enough for developers to use correctly?
- Do developers have access to EFSpecCatalog to look up feature IDs?
- Who maintains the feature ID list that appears in the PR template dropdown?

---

### Decision 2 — Migration Priority: What Confluence Content Goes First?

The gap monitor shows 1,843 pages to migrate. We cannot do all of them at once.

**Options for prioritisation:**

| Option | What migrates first | Rationale |
|---|---|---|
| A — Live features only | Features with `status: live` in EFSpecCatalog | Highest customer-facing risk today |
| B — High-traffic pages | Pages with most Confluence views | Biggest support ticket reduction |
| C — Pilot domain | One domain (e.g., `routing`) end-to-end | Proves the pipeline before scaling |
| D — PO-nominated | POs identify the most-complained-about gaps | Stakeholder buy-in from the start |

**Questions for team:**
- Do we have Confluence page view analytics to identify high-traffic content?
- Which domain has the most complete EFSpecCatalog features to migrate against?
- What is the target: full Confluence decommission, or selective migration only?
- Is there a deadline driving migration priority (e.g., platform cost, audit)?

---

### Decision 3 — Pipeline Scope: Extend DaC or Build a Separate Migration Flow?

Two architectural approaches to handling both new PRs and Confluence migration:

| Option | How it works | Trade-off |
|---|---|---|
| A — Extend DaC | Single pipeline, two triggers: PR events and migration batches | Simpler architecture, but migration may slow PR pipeline |
| B — Separate flow | DaC handles PRs; a parallel migration pipeline handles Confluence | Cleaner separation, but two codebases to maintain |
| C — Sequential | Finish DaC pilot first, build migration flow as Phase 2 | Lower risk, but Confluence gap grows while Phase 1 runs |

**Questions for manager:**
- Is the Confluence migration urgent enough to run in parallel with the DaC pilot?
- What is the engineering capacity available for Phase 1 vs Phase 2 work?
- Is there an organisational deadline for Confluence decommission?

---

### Decision 4 — Canon Axiom Integration: Who Owns Language Quality?

If the pipeline injects axiom claims into generated docs, someone must review whether the
injection is accurate and appropriate. Axioms are authored by the SA team (Andreas Stuber,
co-owner of EFSpecCatalog).

**Questions for team:**
- Does the SA team need to be in the PO review loop specifically for axiom-relevant features?
- Should there be a separate SA review step for docs that reference Canon Axioms?
- Can the PO reviewer be expected to validate axiom language without SA support?
- Is there a way to flag "this doc contains axiom language — SA review recommended"?

---

### Decision 5 — EFSpecCatalog Access and Contribution Model for the DaC Team

The DaC pipeline needs to read EFSpecCatalog (to fetch feature specs at PR time). It may
also need to write (to update `last_referenced_date`, link doc URLs back to features).

**Questions for team:**
- Does the DaC GitHub Action need a dedicated service account with EFSpecCatalog read access?
- Should the DaC pipeline write anything back to EFSpecCatalog, or is it read-only?
- Who from the DaC team needs contributor access to EFSpecCatalog to manage the integration?
- If a feature spec is incomplete (missing `public_description`), does the pipeline fall back
  to diff-only generation or block the doc step? Who is responsible for spec completeness?

---

### Decision 6 — PO Review Redesign: Update the Review Template and SLA?

The current PO review creates a GitHub Issue with a link to the draft doc. With EFSpecCatalog
integration, the Issue content and the review checklist change.

**Questions for team:**
- Should the PO review Issue template be updated now (before pilot) or after pilot proves
  the basic pipeline works?
- What is the right SLA for PO review when the doc is spec-anchored? (Currently 5 business
  days — does this change?)
- Should the PO be expected to update the feature spec if they find errors in it during doc review?
- Who owns the GitHub Issue template updates?

---

### Decision 7 — Coverage Metrics: What Is the Right KPI Going Forward?

The current DaC pipeline tracks coverage by PR. With EFSpecCatalog as source of truth, the
natural coverage unit shifts to features.

**Options:**

| Metric | What it measures | Audience |
|---|---|---|
| PRs with doc coverage | % of merged feature PRs with a published doc | Team leads (delivery) |
| Catalog features with docs | % of EFSpecCatalog features with a published doc | Product + Management |
| Confluence migration burn-down | Gap monitor coverage % (from 0.4% toward 100%) | Management (migration) |
| Customer-visible doc completeness | % of `live` features with `published` docs | External (quality) |

**Questions for manager:**
- Which metric does management care about most for the Monday coverage report?
- Should the gap monitor run automatically as part of the DaC weekly report?
- Is there a target coverage % or a target date that defines "done" for this initiative?

---

### Decision 8 — Phasing: What Gets Built First?

Given the six opportunities identified, the team needs to agree on a Phase 1 scope. Suggested
phasing for discussion:

| Phase | Scope | Dependency |
|---|---|---|
| **Phase 0 (current)** | DaC pilot: PR diff → AI doc → Docusaurus (no EFSpecCatalog connection) | None — already in design |
| **Phase 1** | Feature ID tagging on PRs + spec-grounded generation | Decision 1 resolved |
| **Phase 2** | Status-triggered pipeline events + coverage tracking by feature ID | Phase 1 live |
| **Phase 3** | Confluence migration flow via catalog mapping | Decision 2 and 3 resolved |
| **Phase 4** | Axiom language injection + SA review layer | Decision 4 resolved |

**Questions for manager and team:**
- Should Phase 0 (the pilot) proceed as currently designed, with EFSpecCatalog integration
  as a Phase 1 upgrade? Or should Phase 1 be designed in from the start?
- What is the pilot timeline, and does that constrain how much design change can happen now?
- Who is the decision-maker for phasing — is this a manager call or a team vote?

---

## Summary of Open Decisions

| # | Decision | Owner | Urgency |
|---|---|---|---|
| 1 | How does a PR declare its feature ID? | Team + DaC lead | **Must decide before pilot starts** |
| 2 | What Confluence content migrates first? | Manager + POs | High — gap is growing |
| 3 | Extend DaC pipeline or build separate migration flow? | Manager + tech lead | High |
| 4 | Who owns axiom language quality in published docs? | SA team + PO team | Medium |
| 5 | EFSpecCatalog access model for DaC pipeline | Manager + Andreas/Jawad | High |
| 6 | PO review template and SLA update | DaC lead + POs | Medium |
| 7 | Which coverage metric goes in the Monday report? | Manager | Medium |
| 8 | Phase 0 vs Phase 1 — do we redesign now or upgrade later? | Manager | **Must decide before pilot starts** |

---

## Recommended Meeting Agenda

Suggested structure for the manager and team review session (60 minutes):

| Time | Topic | Goal |
|---|---|---|
| 0–10 min | Context: what changed — EFSpecCatalog as source of truth | Shared understanding |
| 10–20 min | Decision 1: Feature ID tagging mechanism | Closed decision |
| 20–30 min | Decision 8: Phase 0 vs Phase 1 scope | Closed decision |
| 30–40 min | Decision 2 + 3: Migration priority and architecture | Direction set |
| 40–50 min | Decision 5: EFSpecCatalog access model | Owner assigned |
| 50–60 min | Decisions 4, 6, 7: owners assigned, async decisions logged | Task distribution |

---

*Document prepared by Mary (BMAD Strategic Analyst) in collaboration with Navira.*
*EFSpecCatalog reviewed at commit state: 2026-06-24.*
*Next review: after manager/team session outcomes are captured.*
