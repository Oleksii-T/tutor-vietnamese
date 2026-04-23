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

If a word does not fit any existing stage, group it with other unplaced words and **create a new stage** (see Step 4).

---

## Step 3 — Update `curriculum.json`

For each affected stage, append the new Vietnamese words to its `core_vocab` array. Before adding, check if the Vietnamese word is already present. If it is, skip the word entirely — do not update `curriculum.json` or the lesson file for it, and mention it in the final summary as "already exists, skipped".

---

## Step 4 — Update the lesson file for each affected stage

Read the existing lesson file for the stage (path is in `curriculum.json` → `lesson_file`). Make the following additions:

### 4a — Vocabulary table
Add a new row for each new word under the `## Vocabulary` section.

### 4b — Practice sentences
Add sentences that use the new words across all three tiers:

- **Tier 1**: Single word or very short phrase (just the word itself, or a 2-word phrase)
- **Tier 2**: Simple full sentence using one grammar pattern from the stage's grammar notes
- **Tier 3**: Fuller sentence combining the new word with previously learned vocabulary

Aim for at least 1 sentence per tier per new word. Make sure sentences are realistic and grammatically correct Vietnamese.

---

## Step 5 — If a new stage is needed

Only do this if Step 2 identified words that don't fit any existing stage.

### 5a — Add the stage to `curriculum.json`
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

### 5b — Create the lesson file
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
| <example> | <ví dụ> |

### Tier 2 — Simple full sentence, one grammar pattern

| English | Vietnamese |
|---|---|
| <example> | <ví dụ> |

### Tier 3 — Fuller sentence or context-rich phrase

| English | Vietnamese |
|---|---|
| <example> | <ví dụ> |
```

Aim for at least 5 Tier 1, 8 Tier 2, and 8 Tier 3 sentences so the tutor has enough material to run a full 12-sentence session without repeating.

---

## Step 6 — Confirm to the user

After all edits are done, summarise what was changed:

```
Added <N> words:
- Stage 1: word1, word2 → updated curriculum.json + lessons/stage-1-verbs.md
- Stage 6: word3 → updated curriculum.json + lessons/stage-6-descriptive.md
- New Stage 8 "Food & Drink": word4, word5 → created lessons/stage-8-food.md
```

---

## Important rules

- Never modify `progress.json` when adding words — familiarity scores are set during sessions, not upfront.
- Never remove existing words from `core_vocab` or lesson files.
- Always verify Vietnamese is correct before writing — if unsure about a translation, flag it to the user rather than guessing.
- Keep `curriculum.json` and all lesson files in sync at all times.
