# Vietnamese Vocabulary Tutor — Design Spec
_Date: 2026-04-19_

## Overview

A CLI-based adaptive Vietnamese vocabulary tutor that runs entirely inside Claude Code sessions. All state lives in local files. Any Claude session reads these files at startup and immediately knows the learner's full history, current level, weak spots, and what comes next.

Practice direction: **English → Vietnamese** (production, not recognition).

---

## File Structure

```
assistant/
  progress.json          — live state: stage, per-word scores, weak patterns, streak
  curriculum.json        — 7 stages with goals, vocab groups, grammar focus
  vocabulary_raw.csv     — source of truth for all vocabulary (already exists)
  lessons/
    stage-1-verbs.md
    stage-2-numbers-time.md
    stage-3-questions.md
    stage-4-tenses.md
    stage-5-connectors.md
    stage-6-descriptive.md
    stage-7-composition.md
  history/
    YYYY-MM-DD.md        — one file per session day
  docs/
    superpowers/specs/   — design documents
```

Each lesson file contains: the vocab list for that stage, grammar notes specific to that stage, and ~30 sentence templates organized into difficulty tiers 1–3.

---

## Curriculum Stages

Derived from `vocabulary_raw.csv` and existing lesson materials.

| Stage | Name | Focus | Grammar |
|---|---|---|---|
| 1 | Core Verbs & Pronouns | Basic action verbs, I/you/we | "Subject + verb + object" frames |
| 2 | Numbers & Time | Numbers, days, months, o'clock | Time expressions, ordinal patterns |
| 3 | Question Words | khi nào, tại sao, ai, gì, thế nào, ở đâu | Question word placement rules |
| 4 | Tense Markers | đã, đang, sẽ | Past/present/future sentence construction |
| 5 | Connectors | nhưng, và, vì, nên, sau đó, ngoài ra | Compound and causal sentences |
| 6 | Descriptive Vocab | Colors, weather, emotions | Adjective placement, descriptive sentences |
| 7 | Free Composition | All vocab combined | Paragraph-level, complex sentences |

Stage progression is automatic: when average word familiarity in the current stage reaches ≥ 4.0, the next stage unlocks. The learner can also be told explicitly what stage they're on and what comes next.

---

## Progress Tracking (`progress.json`)

```json
{
  "current_stage": 1,
  "sessions_total": 0,
  "streak_days": 0,
  "last_session": null,
  "weak_patterns": [],
  "words": {
    "Đi": { "familiarity": 3, "errors": 1, "last_seen": "2026-04-18" }
  }
}
```

**Familiarity scale (0–5):**
- 0 — unseen
- 1 — introduced, shaky
- 2 — practicing
- 3 — mostly correct
- 4 — solid
- 5 — mastered

**Spaced repetition rule:** Any word with familiarity ≤ 2, or not seen in 5+ days regardless of score, is pulled into the warm-up pool.

**Weak patterns:** Named grammar/category patterns (e.g. "question word position", "tense markers") that Claude flags when 3+ errors share a common cause. These are displayed at session start.

---

## Lesson Flow

### Session start
Claude reads `progress.json`, `curriculum.json`, and the current stage lesson file. Prints:
- Current stage and overall progress
- Streak
- Any weak patterns to watch today
- What new content will be introduced (if any)

### Warm-up (4 sentences)
- Drawn from words with familiarity ≤ 2 or not seen in 5+ days
- Simple sentence frames only (Tier 1)
- Goal: confidence re-entry, spaced repetition

### Main block (8 sentences)
- **Tier 1** (2 sentences): single word or short phrase translation
- **Tier 2** (3 sentences): full simple sentence, one grammar pattern
- **Tier 3** (3 sentences): compound or context-rich sentence

Tier distribution shifts as stage progresses: early sessions in a stage are Tier 1-heavy; later sessions push toward Tier 3.

### Each exchange
1. Claude presents the English sentence
2. Learner types Vietnamese
3. Claude evaluates:
   - **Exact match** → correct, familiarity +1
   - **Close match** (missing tone mark, minor typo) → accepted with correction shown, familiarity +1
   - **Wrong** → explain the rule briefly, show correct answer, familiarity -1 (floor 0), pattern logged

### Session end
- Score: X/12
- Top 1–2 mistakes named explicitly (word or pattern)
- One actionable takeaway line
- `progress.json` updated
- `history/YYYY-MM-DD.md` written with: sentences practiced, answers given, corrections made, score

---

## Adaptive Difficulty

Within a session, difficulty adapts in two ways:

1. **Sentence selection**: if the learner is getting Tier 2 sentences wrong consistently, Claude pulls back to Tier 1 for the remainder of the session.
2. **Stage progression**: blocked until average familiarity ≥ 4.0 for the current stage's core vocab. Claude never skips a stage, but can run extra sessions within a stage if needed.

Words from earlier stages re-appear in later stages to prevent decay. A word mastered (familiarity 5) still surfaces occasionally in warm-ups.

---

## Custom Lessons

Triggered by: *"lesson: weather and seasons"* or *"custom lesson on question words"*.

- Claude pulls matching vocab groups from `vocabulary_raw.csv`
- Runs the full 12-sentence flow (4 warm-up from known vocab + 8 on the topic)
- Logs to `history/YYYY-MM-DD.md` with `custom: true` and topic tag
- Per-word familiarity updates normally — custom lessons feed back into main progression
- `current_stage` in `progress.json` is not advanced by custom lessons

---

## Querying Progress

At any point, the learner can ask:
- *"What step are we on?"* → Claude reads `curriculum.json` + `progress.json` and describes current stage, what's been covered, and what the next stage introduces
- *"What are my weak spots?"* → Claude reads `weak_patterns` and recent history
- *"Show my history"* → Claude summarizes recent `history/` files

---

## Out of Scope

- Audio pronunciation (audio files exist in the repo but are not used by the tutor)
- Vietnamese → English translation direction
- Web or GUI interface
- Multi-user support
