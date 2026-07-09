# hpg-esm-docs Audit — 2026-06-27

## Context

`hpg-esm-docs` is the MkDocs Material site that documents CTSM use on UF's
HiPerGator. Hosted at `https://cdevaneprugh.github.io/hpg-esm-docs/`.

**Audience frame (2026-06-27):** the site serves two readers while the
project is in development:

- **External** — any HiPerGator user wanting to run CTSM. Reads the
  Installation and Running CTSM sections, which use `<group>` placeholders
  and are deliberately generic.
- **Internal (PI + team)** — reads the Research and Swenson Implementation
  sections as project reference. These sections use case-name labels
  (`osbs.swenson.spinup`, `osbs.routing.test`, etc.) as descriptive
  identifiers, which the internal audience navigates directly.

This audit is scoped to freshness, accuracy, and coverage. It does not
argue for a public/private split — case-name labels in the Swenson section
are treated as an established convention.

Last meaningful content commits: 2026-05-19 / 2026-05-20 (Tier 1 / Tier 2
Swenson-section accuracy patches + embedded production plots). Since then,
Phase F closed out, two open scientific questions surfaced, and the
production hillslope file was frozen pending PI investigation.

---

## Overall verdict

The Swenson implementation section is accurate — line numbers, locked
scientific decisions, and the routing-gate framing all resolve correctly.
The gaps are:

1. **Research section** — stale framing ("technical development; custom
   parameters as goal") and obsolete `osbs2.branch.*` reference-cases table.
2. **Coverage gaps** — Phase F results, open scientific questions,
   production-file freeze, post-AD secondary track, and `make_osbs_scrip.py`
   are absent.
3. **Hygiene** — no `CLAUDE.md` or `README.md` at the repo root, two TODO
   plot placeholders in `dataset-comparison.md`, `DOCUMENTATION_TODO.md`
   last touched 2026-01-14.

---

## 1. What's current and correct

| Page | Status |
|---|---|
| `swenson/index.md` | Current to 2026-05-19; routing-gate audit row in dev history |
| `swenson/osbs-implementation.md` | Locked decisions accurate |
| `swenson/hand-binning-and-lake-column.md` | 24-bin scheme, lake column, NWI hole-fill all current (one path-reference fix noted in §2.7) |
| `swenson/lateral-flow-and-routing.md` | All 12 F90 file:line refs verified against CTSM 5.3.085 (`git rev dc3aa5ddc`) |
| `swenson/merit-validation.md` | Regression methodology accurate |
| `swenson/pysheds-porting.md` | Fork setup accurate |
| `research/hillslope.md` | Reflects routing-gate audit; namelist toggles correct |
| Installation, Running CTSM | `<group>` placeholders used correctly |
| Archive section | Labeled archive — hands-off |

---

## 2. Stale or incorrect content

### 2.1 `research/overview.md` lines 101–117 — "Current Status" is stale

Three problems:

- **Line 103:** *"Phase: Technical development (getting model working, not accuracy tuning)"* — outdated. As of 2026-06-27 we are in analysis-and-investigation: Phase F closeout is done; PI is investigating the TAI absence and bridge-zone anomaly.
- **Lines 105–108:** "Hillslope Data: Currently using: Swenson global hillslope dataset (placeholder); Goal: Custom parameters from 1m OSBS LIDAR" — both false. We released the custom NetCDF `hillslopes_osbs_production_c260505.nc` on 2026-05-05 and have it deployed in `osbs.swenson.spinup`.
- **Lines 110–117 "Reference Cases" table:** Lists `osbs2.branch.spillheight/` (retired 2026-04-30), `osbs2.branch.v2/`, `osbs2.branch.v3/` as cases of record. Does not mention `osbs.swenson.spinup` (operative) or `osbs.swenson.post-ad` (secondary). Replace with the current operative + secondary + relevant historical cases.

### 2.2 `research/neon-sites.md` lines 162–168 — duplicated "Reference Cases" table

Same three obsolete `osbs2.branch.*` entries as §2.1, marked "Development / Active / Active". Same fix applies. **Consolidation opportunity:** the two tables are near-duplicates; one canonical location plus a cross-reference from the other page would eliminate the drift risk.

### 2.3 `swenson/dataset-comparison.md` lines 86, 101 — TODO plot placeholders

```html
<!-- TODO: Generate elevation/width profile plots from production data -->
<!-- TODO: Generate column area distribution plots from production data -->
```

The page expects `production_elevation_width.png` and `production_col_areas.png` (which don't exist). Equivalent plots from interim/trimmed datasets do exist (`interior_*`, `trimmed_*`). Either generate the production-stage plots or repoint to the existing interim plots with a caption noting the source dataset.

### 2.4 `swenson/index.md` lines 54–62 — "Current Status" is increasingly stale

*"Analysis is in progress; findings will be documented in a subsequent pass"* (line 60) has been true since 2026-05-19. Five weeks later the actual state is:

- The analysis ran and produced verdicts (see §3.1)
- Production NetCDF is frozen pending PI investigation (see §3.3)
- "Two items remain" list (lines 58–62) doesn't mention the freeze or the PI investigation

### 2.5 `swenson/index.md` Development History table (lines 67–82)

Ends at 2026-05. Add rows for:
- Phase F closeout verdicts (convergence, TAI signal, lake dynamics) — content already drafted in `swenson/STATUS.md` 2026-05-19 change log
- Post-AD continuation run (`osbs.swenson.post-ad`) — 200 yr completed 2026-05-21; idle since
- 2026-06-27 documentation reconciliation + production-file freeze

### 2.6 `DOCUMENTATION_TODO.md`

Repo root file, not served by MkDocs. Last updated 2026-01-14. Most "Planned" items are now done (Glossary, Hillslope Hydrology, NEON Sites, Research Overview, Case Workflow). Outstanding items: Single-Point Runs multi-approach doc, History variables comprehensive list, future Tools & Tips section. Refresh (check off completed items) or retire (move remaining items elsewhere).

### 2.7 `swenson/hand-binning-and-lake-column.md:111` — undefined `osbs4-6` case reference

Line reads:

> To represent the wet, low-lying terrain visible in the NEON LIDAR, the PI installed two SourceMods in `osbs4-6/SourceMods/src.clm/`:

The `osbs4-6` case isn't defined anywhere else in the docs. Even for the PI, this is a stub reference to a historical case that carries no context. The mechanism description ("two SourceMods on `HillslopeHydrologyMod.F90` and `SurfaceWaterMod.F90`") is what the passage actually needs — the path was a label, not a load-bearing pointer. One-line fix, purely for readability.

---

## 3. Coverage gaps — what's missing entirely

### 3.1 Phase F results

The headline scientific findings of the project aren't in the docs site:

- **Convergence PASS** — `drift_50yr = 0.48 %`, well under 3 % threshold
- **TAI signal ABSENT** — `O_SCALAR ≈ 1.0` across the full 25-column × 600-year array. The TAI carbon-side signature (suppressed aerobic decomposition in saturated columns) is not visible in the output. *Most consequential finding of the project to date.*
- **Lake column stable** — max 5.78 m at year 107, drained to 2.5 m by year 600. No runaway; the Darcy-drain mitigation proposed at Phase H is not needed.
- **Bridge-zone anomaly** — chain indices 3–6 (HAND −3 to −1.5 m) have the *deepest* water tables of any lower-hillslope column, despite being closest to the lake. Caused by steep Darcy gradients over short distances.

Plots backing these findings: `swenson/output/2026-05-19_osbs.swenson.spinup_timeseries/` (8 PNGs + h0a/h1a annual NetCDFs).

**Recommended presentation:** new dedicated page `swenson/phase-f-results.md` + copy 2–3 of the most informative PNGs into `docs/swenson/images/` and embed with captions. The internal plot directory pointer can stay in the References section for anyone navigating from the docs to the raw plot files. This gives external readers the visual results; the PI keeps the internal path anchor.

See Decision A for alternatives.

### 3.2 Open scientific questions

The two TAI-relevant questions PI is actively investigating:

1. **O_SCALAR (anoxia) is essentially 1.0 everywhere.** Expected "FZ wet, Upland dry, anoxia depression" pattern does not emerge. Headline issue for the project's central scientific question (TAI carbon dynamics).
2. **Bridge-zone anomaly.** Connects to Phase H B2 (hydraulic conductivity / bin spacing) — now de facto a Phase F follow-up rather than a Phase H prerequisite.

Both belong in `swenson/index.md` as an "Open Questions" section. Framing choice in Decision B.

### 3.3 Production-file freeze callout

Single sentence in `swenson/index.md` Current Status: "The production hillslope file is frozen pending PI investigation of the Phase F TAI absence and bridge-zone anomaly." Consider a matching one-liner in `swenson/osbs-implementation.md` Output Format section for readers who land there directly.

### 3.4 Post-AD case `osbs.swenson.post-ad`

Ran 200 yr successfully (2026-05-20 → 2026-05-21) and has been idle since while PI investigates. Add a row to the Development History table (§2.5) and a mention in Current Status.

### 3.5 `make_osbs_scrip.py` (Phase H Track A SCRIP-file generator)

Active production script in `swenson/scripts/osbs/`. Not mentioned in any docs page. Natural home: `swenson/lateral-flow-and-routing.md` "The mesh-mode workaround" section — currently the section describes what the workaround does but not which script in our repo built the mesh. One-sentence addition with a GitHub URL link.

### 3.6 No `CLAUDE.md` or `README.md` at the docs-repo root

Every other repo in the project (`$BLUE/CLAUDE.md`, `swenson/CLAUDE.md`, `hpg-esm-tools/CLAUDE.md`, the `ctsm5.3` fork) has a `CLAUDE.md`. The docs repo has neither:

- **`CLAUDE.md`** — Claude Code orientation for contributors. Purpose, build/serve commands, navigation philosophy, when to update what. Not served by MkDocs.
- **`README.md`** — GitHub repo landing page. What the site is, the public URL, build/serve commands, contributor pointers.

---

## 4. Recommended route forward — grouped by priority

### Priority 1 — Public-facing stale framing

1. **`research/overview.md` "Current Status"** — rewrite the phase / hillslope-data / reference-cases blocks to reflect the deployed custom NetCDF, current investigation phase, and current cases (`osbs.swenson.spinup` operative; `osbs.swenson.post-ad` secondary; historical `osbs2.branch.*` cases noted as such if kept at all).
2. **`research/neon-sites.md` "Reference Cases"** — same treatment. Consider consolidating to a single canonical location (Decision C).
3. **`swenson/index.md` "Current Status" + Development History table** — add Phase F verdicts summary, production-file freeze callout, post-AD case mention; extend Development History table to 2026-06-27.

### Priority 2 — Coverage gap: Phase F results

4. **New page** `swenson/phase-f-results.md` with the three verdicts, bridge-zone anomaly, and the two open scientific questions. Embed 2–3 key PNGs copied into `docs/swenson/images/`. Add to nav after "Lateral Flow and Routing".

### Priority 3 — Smaller fixes

5. **`swenson/hand-binning-and-lake-column.md:111`** — replace the `osbs4-6/SourceMods/src.clm/` stub path with the mechanism description ("two SourceMods on `HillslopeHydrologyMod.F90` and `SurfaceWaterMod.F90`").
6. **`swenson/dataset-comparison.md`** — resolve the two TODO plot placeholders.
7. **`swenson/lateral-flow-and-routing.md`** — mention `make_osbs_scrip.py` in the mesh-mode workaround section.
8. **`DOCUMENTATION_TODO.md`** — refresh or retire (Decision E).

### Priority 4 — Structural

9. **Add `CLAUDE.md`** at the docs-repo root.
10. **Add `README.md`** at the docs-repo root.

### Not recommended

- Don't touch the Archive section (labeled archive; historical material).
- Don't rewrite Installation or Running CTSM sections (already use `<group>` placeholders correctly).
- Don't refactor case-name labels in the Swenson section — established convention for the internal-audience frame.
- Don't modify file:line references to CTSM source in `lateral-flow-and-routing.md` (all verified accurate).

---

## 5. Open decisions

### Decision A — Phase F results: where to publish?

- **(A1) New dedicated page** `swenson/phase-f-results.md` + embedded plots *(recommended)*.
- (A2) Expand `swenson/lateral-flow-and-routing.md` "Empirical confirmation" section into a longer "Phase F findings" section.
- (A3) Inline summary in `swenson/index.md` Current Status only; defer detailed write-up.

### Decision B — How to frame open scientific questions?

- **(B1) Prominent callout** ("Open Questions" Material admonition) in `swenson/index.md` or the new phase-f-results page *(recommended — transparent about what's unresolved; matches STATUS.md conventions)*.
- (B2) Lower-key mention as part of Current Status.
- (B3) Defer until PI investigation produces a verdict.

### Decision C — Consolidate the duplicated case tables?

- **(C1) Single source** in `research/neon-sites.md`; `research/overview.md` links to it *(recommended — eliminates drift risk)*.
- (C2) Single source in a new `reference/case-inventory.md` page.
- (C3) Keep both tables in sync manually.

### Decision D — Add `CLAUDE.md` and `README.md` to the docs repo root?

- **(D1) Add both** *(recommended — every other repo has both)*.
- (D2) Add `README.md` only.
- (D3) Add neither.

### Decision E — Refresh or retire `DOCUMENTATION_TODO.md`?

- (E1) Refresh (check off completed items, prune retired ones, keep as live backlog).
- (E2) Retire (most items are done; remaining ones move to GitHub issues or stay in your head).

---

## 6. Verification plan

For each edit batch:

1. **Local mkdocs build** — `cd $DOCS && mkdocs serve` to confirm no broken internal links, build cleanly.
2. **Grep for stale anchors** — `grep -rn "osbs2.branch.spillheight" docs/` returns zero after Priority 1.
3. **Cross-check against STATUS** — for any "current state" claim, confirm match against `swenson/STATUS.md` scientific-decisions table and current-state row.
4. **File:line references** — any new F90 / `.py:NNN` reference: `wc -l` the target file and confirm the line matches.
5. **Production-file freeze callout** — search the published site for "frozen" or "freeze" and confirm the production NetCDF's frozen state is mentioned in at least one user-facing page.

---

## Files referenced in this audit

**Read directly:**
- `mkdocs.yml`
- `docs/index.md`, `docs/research/{overview,neon-sites,hillslope}.md`
- `docs/swenson/{index,osbs-implementation,lateral-flow-and-routing,hand-binning-and-lake-column}.md`
- `DOCUMENTATION_TODO.md`

**Grep-scanned for internal-path references** — see §2.7 and the "Not recommended" list under §4.

**Cross-referenced (already in this session's context):**
- `swenson/STATUS.md` (2026-06-27)
- All 8 Swenson phase docs
- `swenson/CLAUDE.md`, `hpg-esm-tools/CLAUDE.md`, `$BLUE/CLAUDE.md`
