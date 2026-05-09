---
name: personal-weekly-review
description: A life-coach agent system for people who feel busy and productive every day but can't tell where their energy and attention actually go — and can't see what's being neglected or whether consistent patterns exist across their life. Surfaces behavioral patterns, emotional undercurrents, and blind spots from daily notes using a layered weekly + quarterly review pipeline with persistent cross-week memory. Use when the user asks to run a weekly review, generate a quarterly review, set up the personal review system, or wants to detect patterns in their daily life and habits.
---

# Personal Weekly Review System

A layered reflection system where raw weekly activity data flows through an agent pipeline to produce weekly reviews, quarterly reflections, and a continuously evolving pattern memory.

## Architecture

```
weekly_agenda.md          ← raw activity log (append-only, never edited)
      ↓
weekly_review_YYYY-Www.md ← one narrative review per week
      ↓
quarterly_review_YYYY-Qn.md ← seasonal reflection, reads weekly reviews only
      ↓
quarterly_pulse_log.md    ← one-line emotional fingerprint per quarter (permanent)
```

`pattern_memory.md` is read **before** and updated **after** every weekly review — it gives the agent multi-week continuity without re-reading all past review files.

---

## Required Files

| File | Purpose |
|---|---|
| `weekly_agenda.md` | Append-only raw activity log. Agent reads but never edits. |
| `pattern_memory.md` | Cross-week behavioral pattern memory. Updated after every weekly review. Max ~60 lines. |
| `quarterly_pulse_log.md` | Permanent one-line-per-quarter emotional record. Created on first quarterly run. |
| `VIBE_LOG.md` | Session retrospectives — bugs, decisions, lessons. Appended conditionally after weekly reviews. |
| `.cursor/rules/weekly_review_agent.mdc` | Agent rule governing weekly review generation. |
| `.cursor/rules/quarterly_review_agent.mdc` | Agent rule governing quarterly review generation. |

---

## Running a Weekly Review

**Trigger phrase:** "run the review" or "summarize the week"

**What the agent does:**
1. Reads `pattern_memory.md` for multi-week context
2. Identifies the most recent `## Weekly Agenda | YYYY-WXX | Date Range` block in `weekly_agenda.md`
3. Drafts the review internally, runs a self-audit (see rule file for audit checklist)
4. Writes `weekly_review_YYYY-Www.md` — never overwrites an existing file
5. Updates `pattern_memory.md` (new patterns, updated counts, resolved entries, pulse log entry)
6. If any non-routine decisions or issues arose this session, appends an entry to `VIBE_LOG.md`

**Review sections (current format, V5+):**
1. The Week's Pulse — 2–4 sentences on the week's underlying vibe, no task recaps
2. Energy & Attention — 1–2 flow moments, 1–2 friction points, synthesised arc (max 5 sentences)
3. Notable Patterns — 2–3 recurring observations
4. The Unspoken Pattern — one thing the author may not have noticed about their own behavior
5. The Coach's Question — one open-ended question to carry into next week

**Language:** Auto-detected from the input. English input → English output. Chinese input → Chinese output.

---

## Running a Quarterly Review

**Trigger phrase:** "run the quarterly review" or "generate the Q[N] review"

**Quarter mapping:**
- Q1 = W01–W13 | Q2 = W14–W26 | Q3 = W27–W39 | Q4 = W40–W52

**What the agent does:**
1. Reads `pattern_memory.md`
2. Reads all available `weekly_review_YYYY-Www.md` files within the quarter's range
3. Notes any missing weeks explicitly — does NOT read `weekly_agenda.md`
4. Writes `quarterly_review_YYYY-Qn.md`
5. Appends one-line pulse entry to `quarterly_pulse_log.md`

---

## Pattern Memory Protocol

After every weekly review:
- **Add** new behavioral patterns spotted this week (with week reference)
- **Update** existing entries with new week counts
- **Move** resolved patterns to the "Recently Resolved" section
- **Append** a one-line entry to the Weekly Pulse Log at the bottom
- Keep total file length under 60 lines

Pattern entries follow this format:
```
- **Pattern name** — description. Observed W03, W05, W07.
```

---

## Key Constraints

- `weekly_agenda.md` is **read-only** — never edit, rewrite, or delete any content in it
- Weekly review files are **never overwritten** — if a file exists, stop and warn
- Quarterly review reads **weekly review files only** — never reaches back to `weekly_agenda.md`
- `quarterly_pulse_log.md` entries are **permanent** — append only, never modified after writing
- `PROJECT_LOG.md` is updated only when the agent instruction file itself changes — not on routine runs

---

## Setting Up From Scratch

If starting in a new project:

1. Create `weekly_agenda.md` with this header format for each week:
   ```
   ## Weekly Agenda | YYYY-WXX | Mon DD – Sun DD
   ```

2. Create `pattern_memory.md` with this skeleton:
   ```
   # Pattern Memory File
   **Purpose:** Cross-week memory. Read before each review, updated after.

   ## Recurring Patterns (Active)

   ---

   ## Recently Resolved / Changed Patterns

   ---

   ## Weekly Pulse Log
   | Week | One-line pulse |
   |---|---|
   ```

3. Copy `.cursor/rules/weekly_review_agent.mdc` and `.cursor/rules/quarterly_review_agent.mdc` into the new project's `.cursor/rules/` folder.

4. `quarterly_pulse_log.md` is created automatically on the first quarterly review run.

---

## Additional Reference

- For full system documentation and design decisions: `README.md`
- For agent instruction version history: `PROJECT_LOG.md`
- For session retrospectives and bugs: `VIBE_LOG.md`
