# HVE Center & Product Studio Implementation Plan

**Status:** Proposed (revised)
**Created:** October 11, 2026
**Revised:** October 12, 2026
**Recommended direction:** Git-backed HVE Center built on microsoft/hve-core, with a lightweight HTML layer on top of the GitHub repository for demos. Decide how to scale to the organization only after partners confirm the approach is useful.

---

## 0. What changed in this revision

| Area | Original plan | Revised plan |
|---|---|---|
| Agent layer | Build custom Context Assembler, DT Coach, PRD Builder, Functional Planner, and GitHub adapter | Reuse the hve-core agents and skills. Build only the context/evidence layer and the UI |
| System of record | New relational database and object storage from Phase 0 | The GitHub repository: markdown artifacts with front-matter, PRs, and Issues/Projects. Add a database only when scaling requires it |
| Approval gates | 8 gates in a custom workflow engine | 4 gates for the demo, implemented as PR reviews, CODEOWNERS, and labels. The 8-gate model stays the target for scale |
| UI | Full authenticated React application | Static HTML layer over the repo for demos. Upgrade path defined in section 11 |
| Delivery approach | Phases 0–6 before teams see value | Demo first (days), validate with partners, then choose a scaling option |
| Optional accelerator | Not considered | hve-squad (multi-agent Squad Coordinator) shown as an optional advanced demo |

---

## 1. Objective

Create a centralized **HVE Center** for Centers of Excellence (CoEs) at ESIs and Software Development Companies (SDCs). Different teams (product, architecture, engineering) work in one place and pass structured, evidence-backed handoffs to each other, from idea to GitHub issue to pull request.

The HVE Center must let teams:

- Maintain reusable company and product context.
- Reference relevant files and repositories.
- Run structured design-thinking exercises with agent assistance.
- Generate grounded, source-linked product artifacts.
- Review and approve decisions before generating engineering handoffs.
- Draft and, after confirmation, publish issues to approved GitHub repositories.

A second goal: the HVE Center and its workshop must be a **reusable template** that partners can run with their own customers.

The new stream must not change, extend, or interrupt the existing Decision Room workshop flow.

---

## 2. Existing assets to reuse

### microsoft/hve-core (supported base)

hve-core is Microsoft's collection of Hypervelocity Engineering components: instructions, agents, and skills for GitHub Copilot. It provides structured Research → Plan → Implement (RPI) workflows and quality gates, and installs from the VS Code Marketplace or as a Copilot CLI plugin.

How the roles in the original plan map to hve-core:

| Role in the original plan | hve-core equivalent | Decision |
|---|---|---|
| DT Coach | `dt-coach` agent (nine Design Thinking methods across problem space, solution space, and validation) | Reuse |
| PRD Builder | Project Planning agents (BRD, PRD, and ADR builders, plus meeting analysis) | Reuse |
| Functional Planner | Functional Planner agent (converts a PRD into a planned hierarchy) | Reuse |
| GitHub adapter | Backlog Manager with `backlog-plan` (read-only) and `backlog-execute` (writes gated by autonomy tiers, dry-run preview, and content sanitization) | Reuse |
| Engineering execution | RPI agent and `rpi-*` phase skills; Copilot coding agent | Reuse |
| Context Assembler / evidence model | Not provided as a product-team capability | **Build (the differentiator)** |
| Visual workspace for non-technical roles | Not provided | **Build (HTML layer)** |

Note: hve-core no longer ships prompt files. Use the current agents and skills, not retired `/` prompt commands.

### hve-squad (optional, community)

hve-squad (Peter-N91/hve-squad) bundles hve-core content into one installable package and adds a **Squad Coordinator** that routes a request to a cast of agents in parallel. It includes a runnable walkthrough of a non-technical product squad (discovery, requirements, roadmap, deck). Related repositories:

- **hve-squad-plugin:** for Copilot CLI and the desktop app.
- **hve-squad-mcp:** an MCP server with Entra authentication, callable from Copilot Studio. Output is advisory text only, with no code execution or deploy.

**Constraints:** hve-squad is a community project, not an official Microsoft repository. The APM install path currently requires APM CLI 0.29.0 (later versions have a dependency regression) and PowerShell 7+. Use it as an "advanced" demo, pre-installed, and never as the foundation partners depend on.

### The Decision Room (existing workshop app)

The Decision Room stays unchanged. In workshops it can serve as the opening, anonymous prioritization activity that picks the challenge the HVE Center then works on.

---

## 3. Options considered

| | **A. HVE Center starter repo** | **B. Teams / Copilot front door** | **C. Full Product Studio (original plan)** | **D. Git-backed Studio (hybrid)** ⭐ |
|---|---|---|---|---|
| **What it is** | Template GitHub repo with hve-core installed, set folder structure (`/context`, `/discovery`, `/decisions`, `/prd`, `/plan`), and handoff templates. Handoffs through PRs and GitHub Issues/Projects | Teams or Copilot Studio agent calling hve-squad-mcp; outputs land as PRs in the starter repo | Custom React app with its own API, database, evidence model, and GitHub App (phases 0–6) | Starter repo (A) as the single source of truth, plus a lightweight UI that reads the repo's markdown and shows gates, traceability, and status |
| **Pros** | Running in days; reuses everything; engineers stay in familiar tools; partners fork per customer | Product owners and analysts never open VS Code; reuses the MCP server with Entra auth | Best business-user experience; full governance and audit; strong differentiating partner IP | Git history gives versioning, audit, and provenance; PR reviews and CODEOWNERS act as approval gates; removes most of Phase 0–1 database work; UI can grow over time |
| **Cons** | Hard for non-technical product roles; no visual workspace; evidence depends on team discipline | Advisory only (no code execution or deploy); needs Azure and Entra deployment; community project, so support risk | Months of work; duplicates hve-core agents, which then drift from upstream; risk of becoming a knowledge-management platform | Permissions set per repo, not per document; large binary uploads need blob storage; still needs some UI work |
| **Best for** | SDC CoEs that are mostly engineers | ESIs with a large business/product side | Partners that want to sell this as a product | **Most CoEs: recommended path** |

**Decision:** start with **D**, delivered first as **A plus a static HTML layer** (section 4). B and C remain scaling options (section 11).

---

## 4. Demo architecture: HTML layer on top of the GitHub repo

### Principle

The GitHub repository is the system of record. The HTML layer is a **read-mostly lens** over it. All writes (artifacts, approvals, issue publication) happen through GitHub, in PRs, Issues, and Copilot agent sessions, so the demo needs no backend, no database, and no tokens in the browser.

### Components

```text
GitHub repo (HVE Center starter kit)
 ├─ .github/                 hve-core agents/skills, CODEOWNERS, PR & issue templates
 ├─ context/                 company.md, product.md, sources/*.md (source register)
 ├─ discovery/               DT exercise outputs (DT Coach), claims with evidence class
 ├─ decisions/               ADR-/DEC-*.md (approved via PR)
 ├─ prd/                     PRD.md (PRD Builder)
 ├─ plan/                    backlog.md (Functional Planner)
 ├─ handoff/                 handoff-contract.md (ready-for-engineering checklist)
 └─ site/                    index.html + app.js + styles.css  (the HTML layer)
        ▲
        │  GitHub Action on push to main: parse front-matter → site/data/index.json
        │  Deploy to GitHub Pages (private Pages / Enterprise for customer data)
```

### Artifact front-matter (the lightweight evidence model)

Every markdown artifact carries front-matter the HTML layer reads:

```yaml
id: DEC-003
type: decision            # source | claim | exercise | decision | prd | plan | handoff
stage: decisions          # context | discovery | decisions | prd | plan | handoff | published
status: approved          # draft | in-review | approved | rejected | deferred
owner: "@product-owner"
evidence: reported        # observed | reported | assumed  (claims/decisions)
confidence: medium
sources: [SRC-002, SRC-005]
derived_from: [EX-001]
links: { pr: 42, issues: [101, 102] }
updated: 2026-10-12
```

### HTML layer screens (demo scope)

1. **HVE Center home:** products/workspaces, stage pipeline (Context → Discovery → Decisions → PRD → Plan → Handoff → Published), and counts per status.
2. **Handoff board:** a Kanban-style view of artifacts by stage and gate status, with open PRs awaiting approval highlighted.
3. **Evidence view:** claims grouped by Observed / Reported / Assumed, with unresolved assumptions and missing evidence highlighted.
4. **Traceability view:** Source → Claim → Decision → Requirement → Issue → PR chain for any item.
5. **Handoff readiness:** the handoff contract checklist scored against the repo (problem, evidence, decisions, open questions, acceptance criteria).
6. **Action links:** "Open in Copilot / VS Code / Codespaces", "Review PR", and "View issue" deep links. No writes happen from the page.

### Why this design for demos

- Built in days with plain HTML/JS. No framework, server, or authentication to build.
- Shows the full product → engineering handoff visually, which non-technical audiences need.
- Fully reusable: partners fork the repo, and the page renders their content automatically.
- Safe: no credentials in the browser, and every write passes through GitHub permissions and PR review.

### Demo limitations (stated openly)

- Read-only UI: product roles still approve in GitHub (PR review UI), not on the page.
- Access control is per repository. Use private repos and private Pages for real customer data.
- Data refreshes on push (via the Action), not in real time.

---

## 5. Non-negotiable boundary

The Decision Room continues to provide:

- Public and anonymous access.
- The existing landing-to-completion stage flow.
- Private attendee commitments.
- Three options generated by one Foundry prompt agent.
- Independent Decision-1 ranking.
- The existing facilitator dashboard and workshop metrics.

The HVE Center must not reuse or modify the workshop stage state in `client/src/App.tsx`, anonymous workshop sessions, workshop case definitions, prompts, API contracts, or dashboard metrics. The HTML layer is a separate static site in a separate repo.

---

## 6. Workflow and approval gates

### Demo gates (4), implemented in GitHub

| Gate | Implementation | Approver |
|---|---|---|
| 1. Context approved | PR adding/updating `context/` merged | Product owner |
| 2. Decisions approved | PR adding `decisions/` merged; front-matter `status: approved` | Product owner + architect |
| 3. PRD approved | PR with `prd/PRD.md` merged | Product owner + engineering lead (CODEOWNERS) |
| 4. Publication confirmed | `backlog-execute` dry-run preview reviewed, then explicit confirmation | Engineering lead |

### Target gates for scale (8, from the original plan)

1. Approve the context snapshot used for an exercise.
2. Approve or revise the problem framing.
3. Select which ideas and conclusions survive.
4. Commit the exercise decisions.
5. Approve the PRD before planning.
6. Approve the functional plan before issue preparation.
7. Approve repository mapping and issue content.
8. Confirm the exact GitHub write operation immediately before publication.

An agent failure, rejected output, or missing approval returns the workflow to the previous editable state. It must not silently continue.

### Workflow states (represented as PR/issue labels in the demo)

```text
DRAFT_CONTEXT → CONTEXT_REVIEW → EXERCISE_ACTIVE → EXERCISE_REVIEW
  → DECISIONS_APPROVED → PRD_DRAFT → PRD_APPROVED → PLAN_DRAFT
  → PLAN_APPROVED → ISSUE_PREVIEW → PUBLICATION_CONFIRMED → PUBLISHED
```

---

## 7. Context and grounding model

The relational model below remains the **target model for scale**. In the demo it is represented by markdown files plus front-matter (section 4).

```text
Organization
  └─ CompanyProfile
      └─ Product
          ├─ ProductProfile
          ├─ Workspace
          │   ├─ Source
          │   ├─ GroundedClaim
          │   ├─ ExerciseRun
          │   ├─ Decision
          │   └─ Artifact
          └─ Integration
              └─ GitHubRepository
```

Every generated claim that may influence a decision should retain: claim text; source identifier and location; source version or retrieval date; evidence classification (`Observed`, `Reported`, `Assumed`); confidence; freshness date; supporting and conflicting sources; human validation status, validating user, and timestamp.

Documents and stakeholder statements remain **reported** evidence. They must not be presented as observed user behavior unless supported by appropriate research.

---

## 8. Design Thinking workshop with GitHub Copilot (SDC)

### Format: "Idea to Issue in 90 minutes" on the partner's own real problem

| Time | Activity | Asset used |
|---|---|---|
| 15 min | Framing: choose one real challenge | Decision Room or whiteboard |
| 30 min | Problem framing → stakeholder map → assumption mapping, with Observed/Reported/Assumed labels visible | hve-core `dt-coach` |
| 15 min | Decision gate: approve/reject items in a PR; show it on the HTML handoff board | GitHub PR + HTML layer |
| 15 min | PRD → functional plan → issue drafts with dry-run preview and human confirmation | PRD Builder, Functional Planner, Backlog Manager |
| 15 min | Payoff: assign one issue to the Copilot coding agent and show the PR | Copilot coding agent |
| Optional | Advanced: the same request through the hve-squad Squad Coordinator | hve-squad (pre-installed) |

### Reusable template partners take away ("HVE Center Starter Kit")

- Starter repo template: folder conventions, front-matter schema, PR/issue templates, and CODEOWNERS per role.
- **Handoff contract:** a one-page "ready for engineering" definition (problem, evidence, decisions, open questions, acceptance criteria).
- Gate checklist mapping approvals to the partner's own roles.
- The HTML layer (fork and deploy to GitHub Pages).
- Facilitator guide and agenda so the partner CoE can run the workshop with their customers.
- Baseline metrics: time from idea to first issue, % of requirements traced to evidence, and rework rate.

---

## 9. Value for partners

- **Repeatable offering:** a packaged "AI-assisted discovery to delivery" engagement ESIs/SDCs can run with their customers, built on maintained Microsoft content.
- **Better product → engineering handoff:** a standard handoff contract reduces vague requirements, the most common delivery failure.
- **Governance from the start:** human gates, evidence labels, and a git audit trail answer "can we trust AI output?"
- **Copilot adoption beyond coding:** brings product owners, analysts, and architects into GitHub Copilot.
- **Clear maturity path:** A → D → (B or C), investing only once value is proven.

---

## 10. Delivery phases (revised)

### Phase 0: Demo kit (target: days)

- Create the HVE Center starter repo with hve-core installed.
- Define folder conventions, front-matter schema, handoff contract, and CODEOWNERS.
- Build the static HTML layer and the GitHub Action that generates `index.json`.
- Seed one sample product end-to-end (context → decisions → PRD → plan → issues).

**Exit criteria:** a facilitator can run the 90-minute workshop end-to-end and show the handoff board and traceability view.

### Phase 1: Partner validation

- Run the workshop with the SDC (and 1–2 more partners).
- Collect feedback: usefulness of the UI for product roles, gate model, handoff contract, metrics.
- Decide on the scaling option (section 11).

**Exit criteria:** documented go/no-go and the chosen scaling path per partner.

### Phase 2+: Scale (only if validated)

The original phases are kept as the scale backlog, re-scoped to avoid rebuilding hve-core:

- Authentication, workspaces, and authorization (original Phase 0–1).
- Interactive exercise runner calling hve-core agents (original Phase 2).
- In-app artifact editing and approval (original Phase 3–4).
- GitHub App with publish center (original Phase 5).
- Multi-user collaboration and advanced grounding (original Phase 6).

---

## 11. Scaling to the organization (to discuss after the demo)

| Path | When to choose | What gets added |
|---|---|---|
| Stay at D (repo + HTML) | Engineering-heavy CoE; small number of products | More templates, more DT exercises, metrics dashboard |
| D + B (Teams/Copilot front door) | Many business/product users who will not use GitHub | Copilot Studio or Teams agent; hve-squad-mcp or equivalent; outputs as PRs |
| D → interactive web app | Product roles must approve and edit in the UI | Authentication (Entra/GitHub OAuth), GitHub App for writes, server-side API |
| C (full Product Studio) | Partner wants to commercialize it as a product | Database, object storage, retrieval, multi-tenancy, full 8-gate engine |

Scale-time requirements carried over from the original plan: GitHub App (not personal access tokens) with least privilege; idempotent, auditable writes; duplicate detection; no automatic edits or closures of existing issues; encryption, tenant isolation, file scanning, retention policies; prompt-injection defenses (sources and repository text are data, never instructions); visible stale or revoked evidence.

---

## 12. Acceptance criteria

### Demo (Phase 0)

- The Decision Room is unchanged.
- The starter repo installs hve-core and runs DT Coach → PRD Builder → Functional Planner → Backlog Manager.
- Every claim and decision artifact has an evidence class, or is marked as an assumption.
- The HTML layer shows pipeline, handoff board, evidence view, and traceability from `index.json`.
- No credentials in the HTML layer. All writes happen through GitHub with review.
- Issue publication happens only after dry-run preview and explicit confirmation.

### Scale (later)

The original first-release criteria apply: authenticated access, workspace isolation, inspectable context snapshots, pause/resume exercises, approvals before PRD and planning, auditable and idempotent external writes, and explicit failure reporting (no success-shaped fallback artifacts).

---

## 13. Key risks

| Risk | Mitigation |
|---|---|
| The new stream leaks into the workshop experience | Separate repo, static site, and deployment |
| Rebuilding what hve-core already provides | Reuse hve-core agents and skills; build only evidence model and UI |
| Upstream hve-core changes break the kit | Pin hve-core version; test before each workshop |
| hve-squad install fails live | Pre-install; pin APM CLI 0.29.0 and PowerShell 7+; treat it as optional |
| Product roles find GitHub too technical | HTML layer for visibility; evaluate Teams/Copilot front door (path B) |
| Customer data exposed via Pages | Private repos and private Pages; sample data for public demos |
| Uploaded content contains prompt injection | Treat sources as data; test adversarial files |
| Generated claims look more certain than their evidence | Mandatory evidence class, confidence, human validation |
| Workflow feels like a long wizard | 4 gates in the demo; direct navigation via the board |
| Agents invent missing requirements | Open-question records in the handoff contract; PR review |
| Duplicate or misplaced GitHub work | `backlog-execute` dry-run preview, repo confirmation, duplicate checks |

---

## 14. Decisions required

**Before the demo:**

1. GitHub organization and demo repo visibility (public sample vs. private).
2. hve-core install channel (extension vs. selective clone) and pinned version.
3. First three DT exercises (proposed: problem framing, assumption mapping, opportunity exploration).
4. Whether hve-squad is shown in the SDC workshop.
5. Sample product/challenge for the seeded walkthrough.

**After partner validation:**

6. Scaling path (section 11).
7. Authentication/identity provider and GitHub App authorization model.
8. Database, object storage, and retrieval technology (only if path C or interactive web app).
9. Artifact formats (Markdown, DOCX, PDF).
10. Data retention, residency, and deletion requirements.

---

## 15. Recommended starting scope

**Start with:** the HVE Center starter repo on hve-core; front-matter evidence model; handoff contract; 4 PR-based gates; static HTML layer (home, handoff board, evidence, traceability, readiness); one seeded end-to-end example; the 90-minute workshop kit.

**Defer:** custom backend and database; authentication; in-app editing; GitHub App; real-time collaboration; portfolio-wide evidence graphs; autonomous publication; changes to existing issues; unreviewed agent-to-agent workflows.

---

## References

- microsoft/hve-core: https://github.com/microsoft/hve-core
- HVE Core documentation: https://microsoft.github.io/hve-core/docs/
- hve-squad (community): https://github.com/Peter-N91/hve-squad (docs: https://peter-n91.github.io/hve-squad/index.html)
- hve-squad-mcp (community): https://github.com/Peter-N91/hve-squad-mcp
