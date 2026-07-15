---
name: ui-flow
description: Worktree-based UI design exploration flow — pure-UI HTML mockups (lo-fi wireframe → per-area lo-fi variants → integrated lo-fi → hi-fi), no app code or backend. Use when the user starts a UI redesign ("/ui-flow start"), wants to split design work into worktrees, asks where the design work stands ("/ui-flow status"), wants to merge a design experiment back, or wants to integrate merged areas into the lo-fi/hi-fi mockup. Also use inside a design worktree to drive the inspiration → ideation → selection → cleanup loop.
---

# UI design exploration flow

A UI redesign runs as **pure UI experimentation in HTML mockups** — no app code, no backend wiring, at any stage. Scoping fixes the key screens and a whole-feature lo-fi wireframe first; the work then splits into independently-decidable areas (e.g. lesson cards, tabs, section structure), each in its own git worktree running the same loop: **inspiration → ideation (lo-fi variants, shown early) → selection → cleanup → merge back**. Integration re-assembles the areas into a whole-feature lo-fi for fast iteration, and only a user-approved lo-fi graduates to the hi-fi mockup. Implementing the design in the real frontend is downstream work, fed by the hi-fi and the decision records — it is not part of this flow. Every phase leaves a written record, and a per-task STATUS.md lets the trunk report progress across all worktrees at any time.

```
dev ──> design/<feature>            (trunk: key screens → wireframe → integration lo-fi → hi-fi)
          ├─ design/<feature>--<task-a>   worktree .design-worktrees/<task-a>
          ├─ design/<feature>--<task-b>   worktree .design-worktrees/<task-b>
          └─ ...                          (each: inspire → ideate lo-fi → select → clean → merge)
```

## Conventions

- **Feature branch:** `design/<feature>` (e.g. `design/learner-dashboard`), cut from fresh `dev`.
- **Task branches:** `design/<feature>--<task>` (e.g. `design/learner-dashboard--lesson-cards`), cut from the feature branch. The `--` separator is required: git forbids `design/learner-dashboard/lesson-cards` once `design/learner-dashboard` exists (ref file/directory conflict).
- **Worktrees:** `.design-worktrees/<task>/` at the repo root. Add `.design-worktrees/` to `.git/info/exclude` (local-only ignore — do not edit the committed `.gitignore`).
- **Trunk worktree (when the main checkout is busy):** if the main checkout carries other active work — another branch, another live session — do **not** switch its branch or commit in it. Give the trunk its own worktree: `git worktree add .design-worktrees/<feature>-trunk design/<feature>`, and run every trunk stage (scaffold, status, merge, integrate) from there. Two sessions sharing one checkout will commit onto each other's branches; this happened once and cost an untangle.
- **Docs (committed on the task branch, merge back with it):**
  ```
  docs/design/<feature>/
  ├── PLAN.md                      # trunk only: key screens, split, integration checklist
  ├── 00-wireframe.html            # whole-feature lo-fi from scoping (every key screen)
  ├── integration/                 # stage 4: lofi.html → (only after lo-fi approval) hifi.html
  ├── renders/                     # screenshots of the final mockups
  └── <task>/
      ├── STATUS.md                # machine-readable progress (format below)
      ├── 01-inspiration.md
      ├── 02-ideation.md
      ├── mockups/                 # lo-fi HTML variants (a.html, b.html…), then final.html
      ├── 03-selection.md
      └── inspiration/             # screenshot images
  ```
- All mockups are **self-contained HTML files** openable straight from disk — no dev server, no build. Lo-fi means grey boxes, real labels, real content shapes, zero styling investment; its job is to be cheap to change.
- Scaffold each task's docs **inside its own worktree**, committed on the task branch — not on the trunk. The trunk only receives a task's docs when the task merges, so there are never stale duplicate copies.

### STATUS.md format

```markdown
---
task: lesson-cards
feature: learner-dashboard
phase: ideation        # inspiration | ideation | selection | cleanup | ready-to-merge | merged
updated: 2026-07-13
blocked: ""            # reason, or empty
decision: ""           # one line, filled in at selection
---

## Now
<one or two lines: what is happening / the next action>

## Log
- 2026-07-13 — started; collected 4 inspiration screenshots
```

Update STATUS.md (phase, updated, Now, Log) **every time a phase completes or the session ends mid-phase**. It is the only channel the trunk reads.

## Detect where you are

If the user passed an argument (`start`, `status`, `merge`, `integrate`), do that stage. Otherwise detect:

- CWD is under `.design-worktrees/` (or `git branch --show-current` matches `design/*--*`) → **task loop**, resume from the phase in STATUS.md.
- On a `design/<feature>` branch in the main checkout → **status**, then ask what's next.
- Anywhere else → **start**.

---

## Stage 1 — Start (`/ui-flow start <feature>`)

1. **Preflight.** `git status` — if the tree is dirty, stop and ask the user what to do with the changes (stash, commit elsewhere, or abort). Never proceed dirty. Also ask whether another session is using this checkout; if the main checkout is busy (or becomes busy mid-flow), switch to the trunk-worktree convention and stop touching the main checkout.
2. **Pull latest.** `git fetch origin dev` (don't check out dev if the main checkout is busy).
3. **Cut the trunk.** `git checkout -b design/<feature>` in the main checkout when it's free, otherwise `git worktree add .design-worktrees/<feature>-trunk -b design/<feature> origin/dev`. If this skill file is untracked, commit it on this branch now — worktrees only see skills committed on their branch.
4. **Grill.** See below. Four things must come out of it before moving on, in this order: the **goal** (what problem this redesign solves), the **user journey** (the full path around the feature), the **key screens list** (chosen from journey moments), and the scope boundaries. Do not propose a split before grilling.
5. **Whole-feature lo-fi wireframe.** Before any split: sketch every key screen in one self-contained HTML file — `docs/design/<feature>/00-wireframe.html`. Grey boxes, real labels, real content shapes; no styling investment. Open it with the user (screenshot for the record) and iterate until the structure is agreed, then commit on the trunk. This wireframe is the map the split is cut from — and the first place layout problems surface, while a fix still costs minutes.
6. **Propose the split.** Present the worktree split **as areas of the wireframe**, each with a one-line scope and a rough size (S/M/L). Get explicit agreement before creating anything.
7. **Scaffold.** Write `docs/design/<feature>/PLAN.md` on the trunk (the goal/problem statement, the key screens, the agreed split as a checklist with one line of scope per task, an empty Integration section) and commit. Then for each task:
   ```bash
   echo ".design-worktrees/" >> .git/info/exclude   # once
   git worktree add .design-worktrees/<task> -b design/<feature>--<task> design/<feature>
   ```
   Always pass the start point (`design/<feature>`) explicitly — defaulting to HEAD once cut worktrees from a branch another session had switched to.
   Inside each worktree, scaffold `docs/design/<feature>/<task>/` (STATUS.md at phase `inspiration`, empty numbered docs, `mockups/` and `inspiration/` dirs) and commit on the task branch.
8. **Hand off.** Tell the user: open a separate Claude Code session in each worktree directory to run its task loop; come back to the trunk and run `/ui-flow status` any time.

### The grill

Style comes from grill-me / grill-with-docs: interview the user **one question at a time**, provide a recommended answer with each question, and if a question can be answered by exploring the codebase, explore instead of asking. When a question hinges on a design tension, name the principle *and its rival* (the user's vocabulary lives in their design-principles memory) and say which wins here.

The goal of this grill is narrower than a full design grill: produce the worktree split. So the questions target **what, exactly, is being redesigned and which decisions are independent**:

- **Goal first: what problem is this redesign solving?** What's important about the current page — what job does it do for the user today, what's failing or missing, and what would success look like? Ask this before any other question; screens, scope, and split are all judged against the goal. Record the answer in PLAN.md as the intent the integration grill re-checks at the end.
- What is wrong with the current UI? (Look at the current components first — e.g. `lms/frontend/src/` — and describe what exists, so the user reacts instead of recalling.)
- Which distinct pieces are in scope? Sweep a checklist, don't rely on what the user volunteers (depth doesn't give coverage): layout/section structure, cards/list items, tabs/navigation, empty/loading/error states, responsive behaviour, interaction flows, copy/microcopy.
- **Map the user journey before choosing any screen.** Walk the full path around the feature end to end — how users discover it, decide, enter, use it day to day, return after absence, finish, and what happens after (for a programme: catalogue → pre-enrolment detail → enrol/pay → dashboard entry → first landing → progressing → deadline nearing → completed → certificate in use). The journey is the coverage sweep for screens — a states-of-the-page question can't surface the moments *around* the page (that's how catalogue discovery got missed in the programme-page round). Mapping the journey does **not** mean building all of it: every journey moment either yields a key screen or gets a conscious out-of-scope entry in PLAN.md. Chosen, not omitted.
- **Which key screens does the feature need?** Choose them from the journey moments this feature owns — pages, states big enough to be their own screen (empty, completed, error), and entry points. The screens list gates everything downstream: the wireframe must cover every key screen, and the split must assign every area of every screen to a worktree or to out-of-scope.
- For each piece: is its design decision independent of the others? Two pieces that must be decided together belong in **one** worktree. A piece with no real decision to make (mechanical restyle) may not need a worktree at all — it can be trunk work.
- What is explicitly out of scope? Write it into PLAN.md.

Splitting rule: **one worktree per independent design decision**, not per component. Keep it to 1–4 worktrees; if the grill surfaces more, the feature is probably two features.

---

## Stage 2 — Task loop (inside a worktree)

Resume from the `phase` in STATUS.md. Complete phases in order; each phase ends by writing its doc and updating STATUS.md.

### Phase 1 — Inspiration → `01-inspiration.md`

Ask the user for screenshots or references (they can paste images into the chat, or drop files into `docs/design/<feature>/<task>/inspiration/`). Read every image. Also screenshot or describe the **current** state of our own UI as reference zero. For each reference, record in `01-inspiration.md`: source, what specifically works about it, and what to steal vs. ignore. Not a moodboard — each entry must say *why it's here*.

### Phase 2 — Ideation → `02-ideation.md`

Variants are **built sequentially as a critique chain, never in parallel**. Parallel generation produces one honest design plus strawmen — the author's first idea absorbs all the real thinking and the rest exist to lose, which makes the selection theatre. The chain forces three genuinely strong contenders:

1. **Variant A** — your best honest design from the inspiration, built fully as a self-contained HTML mockup (`mockups/a.html`), anchored to this area's slice of `00-wireframe.html`. No app code, no backend, opens from disk.
2. **Variant B** — spawn a **fresh sub-agent** (Agent tool) with A's file, the goal from PLAN.md, and the inspiration record. Its brief: critique A — where it's weak against the goal, what it over-privileges, what it ignores — then build `b.html` as a real answer to that critique. B must differ where the critique bites, not cosmetically.
3. **Variant C** — a second fresh sub-agent does the same to B (with A visible as context): critique B, build `c.html`.

Later is not automatically better — each improvement trades something away. The chain's product is three designs that each carry an argument, so selection is a genuine contest. In `02-ideation.md`, record per variant: the critique of its predecessor it answers, the one idea it tests, and what it traded away.

**Show them as they exist**: open each variant in the browser (screenshot via playwright for the record) and walk the user through the chain as it grows — they can stop it early or redirect the next critique. Fill every variant with real content shapes: longest real title, 10 items, the empty state, 390px width.

### Phase 3 — Selection → `03-selection.md`

Grill-style again, one question at a time: force the choice by probing each variant against real content and edge cases (longest title, empty state, 10 items, mobile width). When variants embody a design tension, name the principle and rival. Then record in `03-selection.md`: **the winner, what each rejected variant got right (and whether that gets grafted in), and why — in principle vocabulary**. This is the durable decision record; write it so someone who wasn't there understands the choice.

### Phase 4 — Cleanup

Refine the winner into one clean lo-fi mockup for the area — `mockups/final.html` — folding in whatever the selection grill grafted from the rejected variants. Delete the rejected variant files (git history keeps them). Keep all docs. Set phase to `ready-to-merge`. No app code is written anywhere in this flow — implementation starts only after the integrated hi-fi is approved, as separate work.

### Phase 5 — Merge back

Commit everything on the task branch. Then from the **trunk checkout** (the main checkout when it holds the trunk branch, otherwise `.design-worktrees/<feature>-trunk`):

```bash
git merge design/<feature>--<task>
git worktree remove .design-worktrees/<task>
git branch -d design/<feature>--<task>
```

Before removing the worktree, set its STATUS.md phase to `merged` and commit, so the trunk's merged copy is the final record. Tick the task in PLAN.md.

---

## Stage 3 — Status (`/ui-flow status`)

Run from the trunk checkout (the trunk worktree if one exists). Statuses are read **live from each worktree's filesystem** — no commits or merges needed:

```bash
git worktree list --porcelain | awk '/^worktree /{print $2}'
```

For each worktree path under `.design-worktrees/`, read `<path>/docs/design/<feature>/<task>/STATUS.md`. For tasks whose worktree is gone, read the merged STATUS.md on the trunk. Authority rule: a live worktree's copy always beats the trunk's copy.

Report a table — task, phase, blocked, last updated, one-line "now" — then a short narrative: what's closest to merging, what's blocked and on what, and what the trunk should do next (nothing / merge X / start integration). Flag any STATUS.md older than a few days as possibly stale. Update PLAN.md checkboxes to match reality.

---

## Stage 4 — Integrate (`/ui-flow integrate`)

Only when every task in PLAN.md is merged (run status first to verify). On the trunk, two gates — lo-fi first, hi-fi only after the lo-fi is approved:

1. **Integrated lo-fi.** Assemble every area's `mockups/final.html` into one whole-feature lo-fi covering each key screen: `integration/lofi.html`. Lo-fi is the iteration medium: hunt cross-area drift here (spacing, heading scale, card rhythm, empty-state tone, duplicated CTAs — areas designed apart always disagree somewhere), fix it where a change costs minutes, and loop with the user until they call it OK.
2. **Final grill against the assembled lo-fi.** Does the sum still serve the original intent in PLAN.md? An area decision that no longer holds in context goes back to that area's docs with a note.
3. **Hi-fi — only after the user approves the lo-fi.** Build `integration/hifi.html`: self-contained HTML at design-system fidelity (tokens, spacing, type scale matching the `ui/` system), still pure UI — no app code, no backend. Screenshot every key screen, desktop + 390px, into `renders/`.
4. Fill in the Integration section of PLAN.md: what was assembled, drift fixed, decisions made, open items.
5. **Hand off.** Implementation into the real frontend starts from the hi-fi and the decision records — it is downstream work, not part of this flow.
