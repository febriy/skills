---
name: learning-log
description: Review the current conversation and produce a learning log — key insights, the decisions made and why, the most interesting moment, a quotable line, and one artifact to capture for a LinkedIn or blog post. Use when the user runs /learning-log or asks for a "learning log", "what did I learn today", "blog post material", or a "LinkedIn post".
disable-model-invocation: true
---

# Learning log

Review the current conversation and produce a learning log — key insights, decisions, and one artifact recommendation suitable for a LinkedIn post or blog post.

## Trigger

Run when the user types `/learning-log` or asks for a "learning log", "what did I learn today", "blog post material", or "LinkedIn post".

## Steps

### 1. Conversation Review
Read back through the entire conversation. Look for:
- **Concepts introduced** — new ideas, frameworks, or mental models that came up
- **Decisions made and why** — choices between approaches, and the reasoning behind them
- **Surprising or non-obvious findings** — things that turned out to be more complex, more elegant, or different than expected
- **Patterns and principles** — generalizable lessons that apply beyond this specific task
- **Tools or techniques used** — anything that a practitioner in this field would find useful to know

### 2. Learning Log

Write the learning log in this structure:

---

**Learning Log — [infer a short date-agnostic session title]**

**What we built / worked on:**
One sentence. Not a feature list — the *intent* behind the work.

**Key Learnings:**
3–5 bullet points. Each one should be:
- A concrete, transferable insight (not "we used Puppeteer" but "visual regression loops with Puppeteer let you catch layout bugs before the human ever opens the file")
- Written so a non-participant can understand it
- Something worth knowing — skip obvious steps

**The most interesting thing:**
One paragraph. The single most intellectually interesting moment, tradeoff, or discovery from the session. Write this as if explaining it to a smart friend who wasn't in the room.

**Quotable:**
One punchy sentence that captures a principle from the session. Something worth putting on a slide.

---

### 3. Artifact Recommendation

Recommend exactly ONE artifact the user can capture right now. Choose the one that would make the strongest visual for a post — something that shows the work, not just describes it.

Options to consider:
- A screenshot of a file tree or folder structure that shows a clean architecture
- A code/CSS snippet that demonstrates an elegant pattern
- A side-by-side comparison (e.g., two design variations)
- A diagram or chart generated during the session
- A before/after diff showing a transformation
- A screenshot of a running output (e.g., a rendered HTML page)

Recommendation format:
> **Capture this:** [what to screenshot/copy]
> **Why it works as a post:** [one sentence on why this visual tells a story]
> **Caption suggestion:** [a draft caption for LinkedIn or a blog pull-quote, 1–2 sentences]

### 4. Post Angle (Optional)
If the session has strong post potential, suggest ONE angle for a LinkedIn post or blog post:
- The insight that would resonate most with a technical audience
- The "I didn't expect this" moment
- A "here's how we approached X" framing

Keep this to 3 bullet points max. Don't write the full post — just the angle and opening hook sentence.
