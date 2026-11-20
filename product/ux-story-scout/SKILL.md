---
name: ux-story-scout
description: Runs an impeccable-style UX critique on one learner or admin flow of the Panopticon LMS, turns the gaps into small user stories tagged by risk, area, effort and impact, saves them to Obsidian and re-ranks the whole open backlog with a "Do next" list. Read-only on code. Use only when the user runs /ux-story-scout (it also runs daily from ~/ux-routines).
disable-model-invocation: true
---

# UX story scout

Critique one flow, find small and specific UX gaps, and turn each into a user
story. Then rank every open story and say which to do next. Do not change code.
Do not suggest large redesigns.

Input (optional): a flow name from the list below. Without one, take the next flow
in the rotation.

Output folder: `~/Documents/Obsidian/Panopticon/UX suggestions/`. Create it if it
does not exist. Write only in this folder.

## Flows (rotation order: learner and admin take turns)

| # | Flow | Side | Paths |
|---|---|---|---|
| 1 | Registration and seat claim | Learner | `course/[id]/register/*`, `register/*`, `singpass/error`, seat checks in `useSeatFunding.js` |
| 2 | Courses and learner progress | Admin | `admin/course-management/page.js`, `admin/course-management/[course_id]/` (page, `components/progress`) |
| 3 | Subscription and unsubscribe | Learner | `course/[id]/[course_title]/tools/subscription` |
| 4 | Seat assignments | Admin | `admin/course-management/[course_id]/components/seat-assignments` |
| 5 | Payment results | Learner | `payment/*`, `skillsfuture/*` |
| 6 | Programmes and batches | Admin | `admin/programme-management/**` |
| 7 | Learning loop | Learner | page, assessment, chapter complete, course complete |
| 8 | Organisations | Admin | `admin/organizations/**` (students, seats, groups, administrators, payment advice) |
| 9 | Dashboard and profile | Learner | `dashboard`, `profile` |
| 10 | Course certificates and SkillsFuture | Admin | `admin/course-management/[course_id]/components/certificates`, `components/skillsfuture` |
| 11 | Certificate verify | Learner | `certificates/verify/[certificate_id]` |
| 12 | Users and vouchers | Admin | `admin/users/**`, `admin/vouchers/**` |

Out of scope: the course editor (`course-editor`), `admin/analytics` and
`admin/dashboard` (Metabase).

Paths are under `lms/frontend/src/app/`. Read the code in the current working
directory when it is a checkout of `origin/dev` (the scheduled job makes sure of
this). Otherwise read with `git show origin/dev:<path>`. Never read a feature
branch.

## Steps

1. **Pick the flow.** Read `_scout-log.md` in the output folder. Take the flow
   after the last one logged (after 12, go back to 1), or the flow the user named.
2. **Map the flow.** Spawn an Explore subagent. Ask for each screen in the flow,
   in order: the components, each user action (button, form, link), what the UI
   shows while the action runs, on success and on each error, and the backend
   error messages that can reach the user (with file:line). Also ask, for each
   action that calls the backend:
   - Is the work done before the call returns, or later (a background job, a
     vendor sync)? Is a status stored for the "later" part?
   - Which emails go out (to the learner, the admin or the organisation), and
     from which file?
   - What happens on failure: rollback, Sentry only, retry, refund?
2b. **Screenshots** (only when the input says `screenshots=on`). The app runs at
   `http://localhost:3100`, signed in as one local platform admin who is also
   enrolled in programmes and courses, so both learner and admin pages render.
   The backend at `http://localhost:8099` serves real local data. To find ids for
   URLs, use `curl -s http://localhost:8099/<path>` with the routes from step 2.
   For each screen in the flow (at most 6), run:
   `node /Users/afy/ux-routines/shots/shot.mjs <app path> <YYYYMMDD>-<screen-slug>`
   (exactly this full path; the scheduled run allows only this command form)
   It saves `<name>-desktop.png` and `<name>-mobile.png` into `img/` in the
   output folder and prints the final URL plus failed API calls. If the final
   URL is not the screen you asked for (a redirect), say so; do not retry more
   than once. Open each PNG with Read and look at it. For admin flows, look at
   the desktop PNG only. Error states that local data cannot reach stay
   source-only. Learner pages render with this account's data; an admin-only
   control on a learner page is not a learner finding. With `screenshots=off`,
   skip this step.
3. **Critique (impeccable lens).** Read
   `.claude/skills/impeccable/reference/critique.md` for the heuristics scoring
   guide and the cognitive-load checklist. Then run two independent checks:
   - **A. Design review** (subagent, does not see B): score the 10 Nielsen
     heuristics 0–4 for this flow, check cognitive load (decision points with
     more than 4 options), the emotional journey at high-stakes moments
     (payment, seat claim, assessment submit, an admin delete or bulk action),
     states, and copy. Also apply the checklist below to each user action. For
     an admin flow, also check the screens against the admin UI standards in
     `lms/frontend/CLAUDE.md` (buttons, inputs, modals, destructive previews,
     tables); a breach of a written standard is a finding.
   - **B. Detector** (subagent, does not see A): run
     `.claude/skills/impeccable/scripts/impeccable detect --json <flow folders>`
     on the markup files of the flow. Report counts and real findings; mark
     false positives.
   Give A the screenshot file paths from step 2b, so it judges the rendered
   screens, not only the source. Say in one line at the top of the output file
   what the critique saw: "Live view: stubbed sign-in, local data" or "Source +
   detector only; no live view". Do not ask the user questions and do not save
   impeccable critique snapshots.

   Checklist for each user action:
   - **Feedback:** something changes at once (loading state, disabled button).
   - **Clear outcome:** after success, the user knows what happened and what
     happens next.
   - **Destructive action:** a confirmation names the consequence; an undo or a
     way back exists where it makes sense.
   - **Errors:** each message says why, and what the user can do next. A message
     that tells the user to "contact" someone names who and how.
   - **Dead ends:** no state leaves the user without a next action.
   - **Copy:** plain words; does not narrate what the UI already shows; no
     internal terms.
   - **Access basics:** main actions work by keyboard, have labels, show focus.

   Check one claim from each subagent yourself (open the file:line) before you
   use it.
4. **Respect known facts.** Do not suggest:
   - mobile fixes for admin pages (admin is laptop-only, permanently);
   - an empty state for the learner dashboard (it is never empty);
   - personalisation for AIxTech (it has none);
   - "batch ends" copy (a batch end is a completion deadline, content stays).
5. **Filter.** Keep a finding only if it is visible to a learner or an admin,
   about a day of work or less, and not already in an earlier file in the output
   folder. Also drop a finding when step 2 shows the user already gets the same
   information another way (for example an email after the action). Keep 3–5.
   If fewer qualify, write fewer. Do not pad.
6. **Rate Impact from the whole journey, not the screen alone.** Before you set
   Impact:
   - Use the step 2 answers: what the backend really does, which emails go out,
     what happens on failure. A screen that "says nothing" is not a gap when an
     email or the next screen already says it.
   - Read `~/support-routines/wiki/AIxTech-Support-Runbook.md` if it exists. A
     gap that support already answers by hand, or fixes with manual database or
     vendor work, is at least Med. Cite the runbook section as evidence.
   - For admin flows, count the admins and the learners the action affects: a
     wrong bulk action or seat change hurts many learners at once.
7. **Write the stories** in the user-story-writer format: a short What, then AC
   as one rule per line ("A learner who … sees …", "An admin who … sees …"),
   exact numbers or N with an Open list. Give each story an ID `<YYYYMMDD>-<n>`,
   with `n` one higher than the highest ID already used that day in the folder.
   Under each story add:
   - **Tags:**
     - **Risk:** Low / Med / High — what can break, and one line why
       (for example "touches payment redirect").
     - **Area:** `Learner` or `Admin`, the routes and components it changes,
       then `FE` or `FE+BE`.
     - **Effort:** S (hours) / M (about a day) / L (more than a day — split it
       or drop it).
     - **Impact:** Low / Med / High — who meets this moment, how often, and how
       much it hurts. One line why, based on step 6.
   - **Evidence:** file:line and what the UI does today. When a screenshot shows
     the problem, embed it below: `![[img/<name>-desktop.png|600]]` (or the
     mobile one when the problem is at mobile width, learner pages only).
   - **Heuristic:** the Nielsen heuristic it fixes, and its score from step 3.
   - **User moment:** one line, for example "after clicking Unsubscribe".
8. **Save** to `<YYYY-MM-DD> <flow>.md` in the output folder. Start the file with
   the flow name and side, the commit of `origin/dev` you read, the live-view
   line, the heuristic score table, and a one-line summary. Append one line to
   `_scout-log.md`: date, flow, number of stories, file name.
9. **Re-rank the backlog.** Rewrite `_backlog.md` from all story files in the
   folder. Skip any story whose ID is listed under "Done" or "Dropped" in
   `_backlog.md` (the PM keeps these two lists by hand; copy them over unchanged).
   - Score = Impact (H=3, M=2, L=1) × 2 − Effort (S=1, M=2, L=3) − Risk
     (L=0, M=1, H=2). Ties: higher impact first, then older first.
   - Start with **Do next**: the top 3, each with one line on why now. Prefer
     a mix of areas so two stories do not touch the same component at once.
   - Then one table of all open stories: ID, title, flow, side, Risk, Area,
     Effort, Impact, score, link to the source file (`[[<file name>]]`).
   - Then the "Done" and "Dropped" lists.
10. **Report** in a few lines: the flow, the story titles with their tags, and
    the current Do next top 3.

## Style

ASD-STE100: short sentences, active voice, one word one meaning. No marketing tone.
Name the actor: "the learner", "an admin", not "the user".
