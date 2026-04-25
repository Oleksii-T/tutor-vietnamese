# /add-words — Add New Vocabulary

The user has added new words to `vocabulary_raw.csv` and wants them integrated into the curriculum. Follow every step below in order.

---

## Step 1 — Read the new words

The user will tell you the starting line number. Read `vocabulary_raw.csv` from that line to the end. Extract every row that has both an English and a Vietnamese value. Ignore blank rows.

Format of the file:
```
English,Vietnamese,Column 1
Word,Từ,
```

---

## Step 2 — Assign each word to a stage

Read `curriculum.json` to see all current stages. Use this mapping to decide placement:

| Stage | Name | Assign words that are... |
|---|---|---|
| 1 | Core Verbs & Pronouns | Action verbs, pronouns, basic greetings, basic adjectives |
| 2 | Numbers & Time | Numbers, days of week, months, time words |
| 3 | Question Words | Who / what / where / when / why / how variants |
| 4 | Tense Markers | đã / đang / sẽ / rồi and related temporal particles |
| 5 | Connectors | Conjunctions, connective words (and, but, because, if, so) |
| 6 | Descriptive Vocabulary | Colors, weather, seasons, emotions |
| 7 | Free Composition | No fixed vocab — skip this stage |

If a word clearly belongs to an existing stage, assign it there.

If a word does not fit any existing stage, group it with other unplaced words and **create a new stage** (see Step 7).

---

## Step 3 — Classify each word: core or supporting

For each new word, decide whether it is **core** or **supporting**:

- **Core** — central to the stage's learning goal. The learner must master it to progress.
- **Supporting** — useful context but not essential to the stage goal. Typically nouns or adjectives that appear in sentences but aren't what the stage is teaching.

Use this rule of thumb: if removing the word would leave a gap in the stage's core skill, it's core. If it just enriches a sentence, it's supporting.

---

## Step 4 — Route words into curriculum.json

Read `progress.json` to get `current_stage`.

**For core words**, routing depends on the assigned stage's completion status:

| Assigned stage id | Status | Action |
|---|---|---|
| < current_stage | Completed | Add to that stage's `pending_vocab` as `{"en": "...", "vi": "..."}` |
| == current_stage | In progress | Add to that stage's `core_vocab` array normally |
| > current_stage | Not yet reached | Add to that stage's `core_vocab` array normally |

Before adding to either array, check if the Vietnamese word already exists in `core_vocab` OR `pending_vocab` for that stage. If it does, skip it and note it as "already exists, skipped" in the final summary.

Create `"pending_vocab": []` on the stage entry if the field does not exist yet.

**Supporting words** are not added to `core_vocab` or `pending_vocab` — they only appear in lesson files (Step 5).

---

## Step 5 — Update lesson files

**Core words going to `core_vocab` (active or future stages):**
Add a vocabulary table row + practice sentences (Tiers 1/2/3) to that stage's lesson file.

**Supporting words:**
Add a vocabulary table row + practice sentences to the relevant stage's lesson file, regardless of whether the stage is completed.

**Core words going to `pending_vocab` (completed stages):**
Do NOT touch the original stage's lesson file. These words will get their own lesson file when the Lvl 2 stage is created in Step 6.

### Vocabulary table row
Add under the `## Vocabulary` section.

### Practice sentences
Add sentences using the new words across all three tiers:

- **Tier 1**: Single word or very short phrase
- **Tier 2**: Simple full sentence using one grammar pattern from the stage's grammar notes
- **Tier 3**: Fuller sentence combining the new word with previously learned vocabulary

Aim for at least 1 sentence per tier per new word. Make sure sentences are realistic and grammatically correct Vietnamese.

---

## Step 6 — Check pending pools → create Lvl 2 stages

After all routing is done, check every stage in `curriculum.json` that has a `pending_vocab` array with **4 or more entries**.

For each such stage, create a Lvl 2 stage:

### 6a — Select anchor words from the original stage

Read `progress.json → words` to find familiarity scores for the original stage's `core_vocab`. Sort ascending. Select the N weakest as anchors. If a word has no entry in `progress.json`, treat its familiarity as 0.

| New words in pending_vocab | Anchor count |
|---|---|
| 4–5 | Same count as new words (50 / 50) |
| 6–8 | round(new_count × 0.4) |
| 9+ | round(new_count × 0.3) |

### 6b — Insert the Lvl 2 stage into curriculum.json

1. Increment the `id` of every stage with `id > current_stage` by 1
2. Insert the new stage at position `current_stage + 1` with the incremented id
3. Move all words from `pending_vocab` into the new stage's `core_vocab`; remove `pending_vocab` from the original stage entry

```json
{
  "id": <current_stage + 1>,
  "name": "<Original Stage Name> Lvl 2",
  "goal": "<same skill goal as original, now with expanded vocabulary>",
  "grammar_focus": "<same as original stage>",
  "vocab_groups": ["<same as original stage>"],
  "core_vocab": ["<new vi word 1>", "<new vi word 2>"],
  "anchor_vocab": ["<anchor vi word 1>", "<anchor vi word 2>"],
  "progression_threshold": 4.0,
  "lesson_file": "lessons/<original-slug>-lvl2.md"
}
```

`anchor_vocab` is informational — the tutor does not count anchors toward progression, only `core_vocab`.

### 6c — Create the Lvl 2 lesson file

Create `lessons/<original-slug>-lvl2.md`:

```markdown
# Stage <N>: <Original Name> Lvl 2

## Grammar Notes
<copy from original stage>

## Vocabulary

### New words
| English | Vietnamese |
|---|---|
| <new word> | <new từ> |

### Review anchors (from Stage <original_id>)
| English | Vietnamese |
|---|---|
| <anchor word> | <anchor từ> |

## Sentence Templates

### Tier 1 — Single word or short phrase
(new words only; anchors appear in at most 1–2 sentences)

| English | Vietnamese |
|---|---|

### Tier 2 — Simple full sentence, one grammar pattern
(~60% new-word sentences, ~40% anchor sentences)

| English | Vietnamese |
|---|---|

### Tier 3 — Fuller sentence or context-rich phrase
(~70% new-word sentences, ~30% anchor sentences)

| English | Vietnamese |
|---|---|
```

Aim for at least 5 Tier 1, 8 Tier 2, and 8 Tier 3 sentences so the tutor has enough material to run a full session without repeating.

---

## Step 7 — If a genuinely new category is needed

Only do this if Step 2 identified words that don't fit any existing stage.

### 7a — Add the stage to `curriculum.json`
Append a new entry to the `stages` array:

```json
{
  "id": <next available number>,
  "name": "<descriptive name>",
  "goal": "<one sentence: what the learner can produce after this stage>",
  "grammar_focus": "<key grammar rules relevant to this vocab group>",
  "vocab_groups": ["<group name>"],
  "core_vocab": ["<Vietnamese word 1>", "<Vietnamese word 2>"],
  "progression_threshold": 4.0,
  "lesson_file": "lessons/stage-<N>-<slug>.md"
}
```

### 7b — Create the lesson file
Create `lessons/stage-<N>-<slug>.md` with this structure:

```markdown
# Stage <N>: <Name>

## Grammar Notes

- <Key rule 1>
- <Key rule 2>

## Vocabulary

| English | Vietnamese |
|---|---|
| <word> | <từ> |

## Sentence Templates

### Tier 1 — Single word or short phrase

| English | Vietnamese |
|---|---|

### Tier 2 — Simple full sentence, one grammar pattern

| English | Vietnamese |
|---|---|

### Tier 3 — Fuller sentence or context-rich phrase

| English | Vietnamese |
|---|---|
```

Aim for at least 5 Tier 1, 8 Tier 2, and 8 Tier 3 sentences.

---

## Step 8 — Confirm to the user

After all edits are done, summarise what was changed:

```
Added <N> words:
- Stage 2 (active): word1, word2 → updated core_vocab + lesson file
- Stage 1 (completed): word3 → added to pending_vocab (2/4 needed to trigger Lvl 2)
- Stage 1 (completed): word4, word5 → pending pool hit 4 → created Stage 4 "Core Verbs & Pronouns Lvl 2" with 4 anchors (inserted after current Stage 3)
- New Stage 10 "Food & Drink": word6, word7 → created lesson file
```

---

## Important rules

- Never modify `progress.json` when adding words — familiarity scores are set during sessions, not upfront.
- Never remove existing words from `core_vocab`, `pending_vocab`, or lesson files.
- When renumbering stage ids after a Lvl 2 insertion, do not change `current_stage` in `progress.json` — the user stays on the same stage.
- Lesson file names use descriptive slugs, not stage numbers — so they stay stable even as ids shift.
- Always verify Vietnamese is correct before writing — if unsure about a translation, flag it to the user rather than guessing.
- Keep `curriculum.json` and all lesson files in sync at all times.
