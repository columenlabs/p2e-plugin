# P2E model

Canonical entity and assessment facts. **`p2e-mode` points here** before drafting. Product repos may extend via `docs/P2E-lifecycle.md`.

## Graph

```
Product → Flow (persona | foundation) → Phase → UXO → Layer (Story)
  Layer holds: acceptance criteria, capabilities, relations → other layers
Product → Release → Wave (W{n}, unbounded) → ordered member Layers
```

## Entities

| Entity | Is | Is not |
|--------|----|--------|
| **Product** | Whole map; bound via `.p2e/project.json` → `product_slug` | The git repo |
| **Flow** | Persona (journey) or Foundation (8 immutable platform slots) | A field on a layer |
| **Phase** | Journey step or one Foundation slot | Creatable on Foundation |
| **UXO** | Grouping bucket under a phase (`objectives[]` + `description`) | User-story prose (that is RRR on layers) |
| **Layer** | Landable work under a UXO (RRR, thick-spec, status, wave stamp) | The whole feature |
| **Wave** | First-class package for a Product+Release with unbounded `n` (`W{n}`); gate/branch/status/membership/shipChecks/PR | A fixed W1–W25 enum or a priority rank |
| **Capability** | `INTRODUCES` / `MODIFIES` / `DEPRECATES` (+ `isBreaking`; `DEPRECATES` absorbs retired `REMOVES`) | A UXO objective |
| **Criterion** | One testable AC; verifier and reviewer assess separately | Bulk-approvable |
| **Relation** | `DEPENDS_ON` / `BUILDS_ON` / `FIXES` / `SUPERSEDES` (`FIXES` = layer corrects layer, not a GH bug) | Containment; not a GitHub issue link |

Foundation slots are seeded and immutable. Journey → persona Flow; platform/infra → Foundation.

## Wave (package)

- Unique on `(productId, release, n)` with `n ≥ 1` — **no W25 ceiling**. `W26+` is valid.
- Labels are canonical `W{n}` (legacy `P0`–`P24` aliases may appear on write; storage is `Wn`).
- **Membership** is ordered on the Wave; rewriting members also stamps `Story.wave` / `priority` to `W{n}`.
- A story must not sit in two **OPEN** waves (`DualOpenWave`).
- Freeze fields from `waves.get`: `n`, `gate`, `branch`, `status`, `shipChecks`, `githubPrUrl`, ordered `members`.

**Before BUILD / execute work for a package:** call `waves.get` (by `id` or `release`+`n`) and use its member list — do not invent membership from `stories.list` wave filters alone.

## Layer fields

**RRR:** `storyAs`, `storyWant`, `storySoThat`, `background`.

**Thick-spec:** `filesHint`, `constraints`, `nonGoals`, `contextDocs`, `effortHint`, `verificationCmd`. Thick = all six set. Thick gate (`validate op=run`) before `OPEN → IN_PROGRESS`.

**Sizing** (`XS`–`XXL`) and **wave** (`W{n}` / `null`) are independent. Wave is the package stamp; open-work ordering within a release follows Wave `n` ascending, then oldest-first.

## Status vs AC blocked

- **`StoryStatus.BLOCKED`** — unfinished dependency (`DEPENDS_ON`) or equivalent wait.
- **AC verdict `BLOCKED`** — coder and verifier cannot align on that criterion; escalate to human.

Statuses: `DRAFT | OPEN | BLOCKED | IN_PROGRESS | IN_REVIEW | DONE | CANCELLED`.

## Assessments

Role ladder: **coder** → **verifier** → **reviewer** → **human**.

| Role | MCP usage |
|------|-----------|
| verifier | `criteria op=propose` with verifier role; `criteria op=list` includes verifier block |
| reviewer | `criteria op=propose` with reviewer role; `criteria op=list` with reviewer viewer role (verifier block omitted — blind invariant) |

Wire enum values for verifier and reviewer roles: read the live **`criteria`** tool schema at session start — MCP is authoritative. Plugin docs use **reviewer** only; never the retired role name.

Verdicts: verifier `PASS | FAIL | BLOCKED`; reviewer `PASS | FAIL` only. Absence / `NOT_TESTED` = unassessed.
- Agents write via `criteria op=propose`. `op=verdict` / `op=toggle` are not for agents.
- Reviewer is blind to verifier output — `criteria op=list` with reviewer viewer role (table above; verifier block omitted).
- Evidence attaches via `story_assets` (`criterion_id`); proof markdown via `evidence` tool where applicable.

## Tag shapes

- **backend** — automated/unit proof; digest/`ac{N}-proof.md` expected.
- **ui** — visual proof (screenshot/video); digest alone is insufficient.
- **external** — contract/integration proof; digest/`ac{N}-proof.md` expected.
- **docs** — short written note sufficient.
- **security** — security review expected in addition to other tags' shapes.

Multi-tag layers take the union of shapes.

## UXO facts

`objectives[]` are MECE noun phrases within the UXO; `description` synthesizes them. Layers land on objectives; gaps are new layers, not diluted objectives.

## Stories vs Issues

**Separation of concerns is primary.** Do not treat GitHub Issues and P2E layers as mirrors.

| Track in | For | Not for |
|----------|-----|---------|
| **P2E Layer (Story)** | Work that **introduces / modifies / deprecates** capabilities under a UXO (Layer → UXO → Phase → Flow → Product) | Ordinary bugs that only repair existing behavior without a capability change |
| **GitHub Issue** | Usually **bugs** (and similar defects) on an already-shipped feature/capability — tracked in the repo's issue tracker | Capability evolution that belongs on the map as a new or corrective layer |

### When to create which

- **Create a P2E layer** when the change is product work on the map: new capability, intentional modify/deprecate, thick-spec BUILD item, Wave membership.
- **Create a GitHub issue** when something is broken on an existing capability and the fix does **not** redefine the capability story — subscribe/manage issues in GitHub independently of P2E releases.
- **Do not** default to `create_github_issue` / `sync_github_status` / issue↔story body sync. Historical GH↔story coupling is retired as the desired operating model; harnesses no longer need it.
- Optional one-way links (e.g. a PR that mentions an issue) are fine; **do not** auto-close GH issues from story DONE, and **do not** mutate live map stories from issue webhooks, unless a migration doc explicitly asks for a one-time note.

### `FIXES` relations (layers only)

`Relation.type = FIXES` links one **layer** that intentionally corrects another **layer's** capability story. It is **not** a stand-in for "this GitHub bug." Bugs stay in GitHub; capability corrections stay as layers (often with `FIXES` + `MODIFIES` / `INTRODUCES` change entries).

### Why

Project-scoped **subscriptions** can manage **issues from GitHub** and **releases / Waves from P2E** independently.

## Invariants

- MCP is authoritative — no parallel story state in files.
- Preview before writes on layers/UXOs/criteria/capabilities.
- Never create Foundation phases via MCP.
- Human Mark DONE is the sole acceptance gate for a layer.
- Stories (layers) ≠ GitHub Issues — see [Stories vs Issues](#stories-vs-issues); no default issue↔story sync.
- Wave `n` is unbounded — never reject or clamp at W25.
