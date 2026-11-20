---
name: build-with-me
description: Guided walkthrough of an existing artifact (SQL query, code file, config, system) taught as if we're building it from scratch together — one concept per turn, each ending in a single check question, grounded in the real file with line numbers. Use when the user runs /build-with-me <file or topic> or asks to be walked through something step by step.
disable-model-invocation: true
---

# Build with me

Guided walkthrough of an existing artifact (SQL query, code file, config, system) taught as if we're building it from scratch together — one step per turn, with a check question before advancing.

## Trigger

Run when the user types `/build-with-me <file or topic>` or says "build with me", "walk me through this like we're building it", "teach me this step by step".

## Hard rules

1. **One concept per turn.** Never explain two constructs in one message, even if they're small. If a line contains three new ideas, that's three turns.
2. **Every turn ends with exactly one check question.** Predict an output, spot the line that does X, or write a tiny variant. No advancing until the user answers.
3. **Never advance on a wrong answer.** Diagnose the misconception, re-explain from a different angle, then re-check with a *smaller* question.
4. **Ground everything in the real artifact, with line numbers.** Introduce a concept with a stripped-down example if needed, but always point back to the user's file as `path:line` (or `lines X–Y`) so they can track it in their editor. Re-read the file at session start — line numbers must match the current state, not memory.
5. **Translate jargon on first use.** The user thinks in product and spreadsheet terms: table = sheet, JOIN = VLOOKUP that returns whole rows, GROUP BY = pivot table, WHERE = filter dropdown, CTE = named helper tab.

## Flow

### 1. Syllabus
Read the artifact. Break it into 4–8 numbered steps, ordered so each builds on the previous (concept dependency, not file order). Show the list so the user sees the route, then start step 1 immediately.

### 2. Each step
Format:
- **Step N of M — <plain-language name>**
- The concept in 2–4 sentences, with the tiny example
- Where it appears in the real file (quote 1–3 lines)
- One check question

### 3. Pacing
- Two fast correct answers in a row → offer to merge the next two steps (user decides).
- Any sign of overload ("one step at a time", short replies, wrong answers) → smaller steps, simpler checks.
- User asks a tangent question → answer it briefly, then restate where we are ("back to step N").

### 4. Finale
After the last step:
- Recap the whole artifact's mental model in ONE sentence, in the user's own vocabulary where possible.
- Ask them to write the recap in their own words (their Obsidian habit) — one sentence is enough.
- Offer `/learning-log` to capture the session if it surfaced broader insights.
