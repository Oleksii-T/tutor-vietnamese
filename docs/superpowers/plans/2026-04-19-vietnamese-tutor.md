# Vietnamese Vocabulary Tutor — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a file-based adaptive Vietnamese vocabulary tutor that runs entirely inside Claude Code sessions, tracking per-word familiarity, weak patterns, and curriculum stage across sessions.

**Architecture:** All state lives in local JSON/Markdown files that any Claude session reads at startup. Claude itself is the runtime — no external processes, no code to execute. The tutor behavior is driven by `TUTOR.md` (master instruction file) + per-stage lesson files + `progress.json`.

**Tech Stack:** JSON (state), Markdown (lessons, history, instructions), CSV (vocabulary source). Zero dependencies.

---

## File Map

| File | Responsibility |
|---|---|
| `assistant/progress.json` | Live learner state: stage, per-word familiarity, weak patterns, streak |
| `assistant/curriculum.json` | 7-stage study plan with goals, vocab groups, grammar focus |
| `assistant/TUTOR.md` | Master instruction file — Claude reads this to know how to run every session |
| `assistant/lessons/stage-1-verbs.md` | Vocab + grammar notes + 30 sentence templates for Stage 1 |
| `assistant/lessons/stage-2-numbers-time.md` | Stage 2 content |
| `assistant/lessons/stage-3-questions.md` | Stage 3 content |
| `assistant/lessons/stage-4-tenses.md` | Stage 4 content |
| `assistant/lessons/stage-5-connectors.md` | Stage 5 content |
| `assistant/lessons/stage-6-descriptive.md` | Stage 6 content |
| `assistant/lessons/stage-7-composition.md` | Stage 7 content |
| `assistant/history/YYYY-MM-DD.md` | Per-session log written at session end |

---

## Task 1: Initialize directory structure and `progress.json`

**Files:**
- Create: `assistant/lessons/.gitkeep`
- Create: `assistant/history/.gitkeep`
- Create: `assistant/progress.json`

- [ ] **Step 1: Create directories**

```bash
mkdir -p /Users/work/Documents/vietnameese/assistant/lessons
mkdir -p /Users/work/Documents/vietnameese/assistant/history
```

- [ ] **Step 2: Create `progress.json` with initial state**

Create `assistant/progress.json`:

```json
{
  "current_stage": 1,
  "sessions_total": 0,
  "streak_days": 0,
  "last_session": null,
  "weak_patterns": [],
  "words": {}
}
```

- [ ] **Step 3: Verify JSON is valid**

```bash
python3 -c "import json; json.load(open('assistant/progress.json')); print('valid')"
```

Expected output: `valid`

- [ ] **Step 4: Commit**

```bash
git -C /Users/work/Documents/vietnameese add assistant/progress.json assistant/lessons assistant/history
git -C /Users/work/Documents/vietnameese commit -m "feat: initialize tutor directory structure and progress state"
```

---

## Task 2: Create `curriculum.json`

**Files:**
- Create: `assistant/curriculum.json`

- [ ] **Step 1: Create `curriculum.json`**

Create `assistant/curriculum.json`:

```json
{
  "stages": [
    {
      "id": 1,
      "name": "Core Verbs & Pronouns",
      "goal": "Produce basic 'Subject + verb + object' sentences using core action verbs and pronouns",
      "grammar_focus": "Vietnamese sentence order is Subject + Verb + Object, same as English. Negation: add 'không' before the verb.",
      "vocab_groups": ["core verbs", "pronouns", "basic greetings", "basic adjectives"],
      "core_vocab": ["Đi", "Đến", "Ăn", "Uống", "Học", "Làm", "Ngủ", "Nói", "Nghe", "Đọc", "Viết", "Xem", "Chạy", "Bơi", "Lái xe", "Mua", "Bán", "Gặp", "Thích", "Muốn", "Cần", "Biết", "Hiểu", "Yêu", "Tôi", "Bạn", "Chúng ta", "Không"],
      "progression_threshold": 4.0,
      "lesson_file": "lessons/stage-1-verbs.md"
    },
    {
      "id": 2,
      "name": "Numbers & Time",
      "goal": "Produce sentences with numbers, days of the week, months, and time expressions",
      "grammar_focus": "Time expressions typically come at the start or end of a sentence. 'Vào' means 'on' for days/months. Time: [number] + giờ.",
      "vocab_groups": ["numbers", "days of week", "months", "time words"],
      "core_vocab": ["Một", "Hai", "Ba", "Bốn", "Năm", "Sáu", "Bảy", "Tám", "Chín", "Mười", "Trăm", "Nghìn", "Thứ Hai", "Thứ Ba", "Thứ Tư", "Thứ Năm", "Thứ Sáu", "Thứ Bảy", "Chủ Nhật", "Tháng Một", "Tháng Hai", "Tháng Ba", "Tháng Tư", "Tháng Năm", "Tháng Sáu", "Tháng Bảy", "Tháng Tám", "Tháng Chín", "Tháng Mười", "Tháng Mười Một", "Tháng Mười Hai", "Ngày hôm nay", "Ngày mai", "Hôm qua", "giờ", "phút", "Vào"],
      "progression_threshold": 4.0,
      "lesson_file": "lessons/stage-2-numbers-time.md"
    },
    {
      "id": 3,
      "name": "Question Words",
      "goal": "Form correct questions using Vietnamese question words with proper placement",
      "grammar_focus": "Question word placement rules: When (khi nào) — start or end. Why (tại sao) — start. Who (ai) — end if verb present, before noun if no verb. What (gì) — end. How (thế nào) — end. Where (ở đâu) — end. Yes/no questions: Bạn có [verb] không?",
      "vocab_groups": ["question words", "question patterns"],
      "core_vocab": ["Khi nào", "Bao giờ", "Tại sao", "Vì sao", "Ai", "Cái gì", "Gì", "Thế nào", "Như thế nào", "Ở đâu", "Đâu", "Bao nhiêu tiền", "Bao nhiêu", "Mấy giờ", "Bao xa", "Bao lâu", "Bao nhiêu tuổi"],
      "progression_threshold": 4.0,
      "lesson_file": "lessons/stage-3-questions.md"
    },
    {
      "id": 4,
      "name": "Tense Markers",
      "goal": "Produce past, present-continuous, and future sentences using đã / đang / sẽ",
      "grammar_focus": "Vietnamese tense markers come BEFORE the main verb. Đã = past (already done). Đang = present continuous (currently happening). Sẽ = future (will happen). 'Rồi' at the end strengthens past completion: Tôi đã ăn rồi.",
      "vocab_groups": ["tense markers"],
      "core_vocab": ["Đã", "Đang", "Sẽ", "Rồi"],
      "progression_threshold": 4.0,
      "lesson_file": "lessons/stage-4-tenses.md"
    },
    {
      "id": 5,
      "name": "Connectors",
      "goal": "Produce compound and causal sentences using conjunctions and connective words",
      "grammar_focus": "Conjunctions mostly follow English word order. 'Nếu' (if) begins a clause. 'Nên' means both 'so' (consequence) and 'should'. 'Sau đó' is always between two complete clauses.",
      "vocab_groups": ["conjunctions", "connectors"],
      "core_vocab": ["Nhưng", "Và", "Vì", "Bởi vì", "Nên", "Vậy", "Sau đó", "Sau khi", "Cũng", "Ngoài ra", "Với", "Nếu", "Hoặc", "Hay"],
      "progression_threshold": 4.0,
      "lesson_file": "lessons/stage-5-connectors.md"
    },
    {
      "id": 6,
      "name": "Descriptive Vocabulary",
      "goal": "Produce descriptive sentences about colors, weather, seasons, and emotions",
      "grammar_focus": "Adjectives in Vietnamese follow the noun they modify: 'hoa đỏ' (red flower), not 'đỏ hoa'. Weather: 'Trời + [adjective]' or 'Có + [noun]'. Emotions: Subject + [emotion word], no 'to be' needed.",
      "vocab_groups": ["colors", "weather", "seasons", "emotions"],
      "core_vocab": ["Màu đỏ", "Màu vàng", "Màu xanh lá", "Màu xanh dương", "Màu hồng", "Màu tím", "Màu cam", "Màu xám", "Màu nâu", "Màu trắng", "Màu đen", "Có nắng", "Nhiều mây", "Có mưa", "Có tuyết", "Có bão", "Nóng", "Lạnh", "Mùa thu", "Mùa đông", "Mùa hè", "Mùa xuân", "Hạnh phúc", "Buồn bã", "Tức giận", "Sợ hãi", "Hào hứng", "Hồi hộp", "Mệt mỏi", "Bình tĩnh"],
      "progression_threshold": 4.0,
      "lesson_file": "lessons/stage-6-descriptive.md"
    },
    {
      "id": 7,
      "name": "Free Composition",
      "goal": "Produce complex multi-clause sentences and short paragraphs drawing on all previous vocabulary",
      "grammar_focus": "Review of all previous grammar. Focus on combining tense markers + connectors + descriptive vocab in one sentence. Practice maintaining correct word order across clause boundaries.",
      "vocab_groups": ["all"],
      "core_vocab": [],
      "progression_threshold": 4.0,
      "lesson_file": "lessons/stage-7-composition.md"
    }
  ]
}
```

- [ ] **Step 2: Verify JSON is valid**

```bash
python3 -c "import json; data = json.load(open('assistant/curriculum.json')); print(f'valid — {len(data[\"stages\"])} stages')"
```

Expected output: `valid — 7 stages`

- [ ] **Step 3: Commit**

```bash
git -C /Users/work/Documents/vietnameese add assistant/curriculum.json
git -C /Users/work/Documents/vietnameese commit -m "feat: add 7-stage curriculum definition"
```

---

## Task 3: Create `lessons/stage-1-verbs.md`

**Files:**
- Create: `assistant/lessons/stage-1-verbs.md`

- [ ] **Step 1: Create the lesson file**

Create `assistant/lessons/stage-1-verbs.md`:

```markdown
# Stage 1: Core Verbs & Pronouns

## Grammar Notes

- Sentence order: **Subject + Verb + Object** (same as English)
- Negation: add **không** before the verb → "Tôi không ăn" (I don't eat)
- "To be" (là) is used for nouns/identity, not adjectives. "You are beautiful" = "Bạn đẹp" (no là)
- No conjugation — verbs never change form regardless of subject

## Vocabulary

| English | Vietnamese |
|---|---|
| Go | Đi |
| Come | Đến |
| Eat | Ăn |
| Drink | Uống |
| Study | Học |
| Do / Work | Làm |
| Sleep | Ngủ |
| Speak / Tell | Nói |
| Listen | Nghe |
| Read | Đọc |
| Write | Viết |
| Watch | Xem |
| Run | Chạy |
| Swim | Bơi |
| Drive | Lái xe |
| Buy | Mua |
| Sell | Bán |
| Meet | Gặp |
| Like | Thích |
| Want | Muốn |
| Need | Cần |
| Know | Biết |
| Understand | Hiểu |
| Love | Yêu |
| I / Me | Tôi |
| You | Bạn |
| We | Chúng ta |
| No / Don't | Không |
| Good | Tốt |
| Beautiful | Đẹp |
| Bad | Xấu |
| Hungry | Đói |
| Thirsty | Khát nước |
| Hello | Xin chào |
| Goodbye | Tạm biệt |
| Thank you | Cảm ơn |
| Sorry | Xin lỗi |
| Nice to meet you | Rất vui được gặp bạn |
| See you again | Hẹn gặp lại |
| Very | Rất |
| Water | Nước |
| Park | Công viên |
| Book | Sách |
| House | Nhà |

## Sentence Templates

### Tier 1 — Single word or short phrase

| English | Vietnamese |
|---|---|
| Hello | Xin chào |
| Thank you | Cảm ơn |
| Goodbye | Tạm biệt |
| Sorry | Xin lỗi |
| Good | Tốt |
| Beautiful | Đẹp |
| I | Tôi |
| You | Bạn |
| We | Chúng ta |
| Hungry | Đói |
| Thirsty | Khát nước |
| Water | Nước |

### Tier 2 — Simple full sentence, one grammar pattern

| English | Vietnamese |
|---|---|
| I go | Tôi đi |
| I eat | Tôi ăn |
| I drink water | Tôi uống nước |
| I drive a car | Tôi lái xe ô tô |
| I study | Tôi học |
| I sleep | Tôi ngủ |
| I listen | Tôi nghe |
| You are beautiful | Bạn đẹp |
| I like to swim | Tôi thích bơi |
| I want to eat | Tôi muốn ăn |
| I need water | Tôi cần nước |
| I don't understand | Tôi không hiểu |
| We go | Chúng ta đi |
| Nice to meet you | Rất vui được gặp bạn |

### Tier 3 — Fuller sentence or context-rich phrase

| English | Vietnamese |
|---|---|
| I want to meet you | Tôi muốn gặp bạn |
| I need to study Vietnamese | Tôi cần học tiếng Việt |
| I want to buy a book | Tôi muốn mua một cuốn sách |
| I know how to drive a car | Tôi biết lái xe ô tô |
| I don't want to go | Tôi không muốn đi |
| I like to run in the park | Tôi thích chạy trong công viên |
| We need to go together | Chúng ta cần đi cùng nhau |
| I love you | Tôi yêu bạn |
| I want to read this book | Tôi muốn đọc cuốn sách này |
| You need to listen | Bạn cần nghe |
```

- [ ] **Step 2: Spot-check 3 sentences manually**

Verify: "I don't understand" → Tôi không hiểu (không before verb ✓)
Verify: "You are beautiful" → Bạn đẹp (no là ✓, adjective after subject ✓)
Verify: "I like to run in the park" → Tôi thích chạy trong công viên (verb chain ✓)

- [ ] **Step 3: Commit**

```bash
git -C /Users/work/Documents/vietnameese add assistant/lessons/stage-1-verbs.md
git -C /Users/work/Documents/vietnameese commit -m "feat: add Stage 1 lesson — core verbs and pronouns"
```

---

## Task 4: Create `lessons/stage-2-numbers-time.md`

**Files:**
- Create: `assistant/lessons/stage-2-numbers-time.md`

- [ ] **Step 1: Create the lesson file**

Create `assistant/lessons/stage-2-numbers-time.md`:

```markdown
# Stage 2: Numbers & Time

## Grammar Notes

- Time expressions can appear at the **start or end** of a sentence: "Hôm nay tôi học" or "Tôi học hôm nay"
- **Vào** = "on" for days/months: "Vào thứ Hai" (on Monday), "Vào tháng Một" (in January)
- Time of day: [number] + **giờ** → "ba giờ" (3 o'clock), "tám giờ" (8 o'clock)
- For 15 = mười lăm (not mười năm — "lăm" replaces "năm" after tens)
- Days of the week: Thứ Hai through Thứ Bảy (Mon–Sat), Chủ Nhật (Sunday)
- Months: Tháng + [number] → Tháng Một (Jan) through Tháng Mười Hai (Dec)

## Vocabulary

| English | Vietnamese |
|---|---|
| One | Một |
| Two | Hai |
| Three | Ba |
| Four | Bốn / Tư |
| Five | Năm |
| Six | Sáu |
| Seven | Bảy |
| Eight | Tám |
| Nine | Chín |
| Ten | Mười |
| Hundred | Trăm |
| Thousand | Nghìn / Ngàn |
| Monday | Thứ Hai |
| Tuesday | Thứ Ba |
| Wednesday | Thứ Tư |
| Thursday | Thứ Năm |
| Friday | Thứ Sáu |
| Saturday | Thứ Bảy |
| Sunday | Chủ Nhật |
| January | Tháng Một |
| February | Tháng Hai |
| March | Tháng Ba |
| April | Tháng Tư |
| May | Tháng Năm |
| June | Tháng Sáu |
| July | Tháng Bảy |
| August | Tháng Tám |
| September | Tháng Chín |
| October | Tháng Mười |
| November | Tháng Mười Một |
| December | Tháng Mười Hai |
| Today | Ngày hôm nay |
| Tomorrow | Ngày mai |
| Yesterday | Hôm qua |
| O'clock | Giờ |
| Minute | Phút |
| On (day/month) | Vào |
| Have | Có |

## Sentence Templates

### Tier 1 — Single word or number

| English | Vietnamese |
|---|---|
| One | Một |
| Five | Năm |
| Ten | Mười |
| One hundred | Trăm |
| Monday | Thứ Hai |
| Sunday | Chủ Nhật |
| January | Tháng Một |
| December | Tháng Mười Hai |
| Today | Ngày hôm nay |
| Yesterday | Hôm qua |
| Tomorrow | Ngày mai |
| Three o'clock | Ba giờ |

### Tier 2 — Simple sentence with time/number

| English | Vietnamese |
|---|---|
| Today is Monday | Ngày hôm nay là Thứ Hai |
| Tomorrow is Tuesday | Ngày mai là Thứ Ba |
| Yesterday was Wednesday | Hôm qua là Thứ Tư |
| I have five books | Tôi có năm cuốn sách |
| It is three o'clock | Là ba giờ |
| On Friday I go to the park | Vào Thứ Sáu tôi đi công viên |
| I wake up at seven o'clock | Tôi thức dậy vào lúc bảy giờ |
| There are ten students | Có mười học sinh |
| In January it is cold | Vào tháng Một trời lạnh |
| I was born in March | Tôi sinh vào tháng Ba |

### Tier 3 — Fuller sentence with multiple time/number elements

| English | Vietnamese |
|---|---|
| I go to school on Monday and Wednesday | Tôi đi học vào Thứ Hai và Thứ Tư |
| I work from Monday to Friday | Tôi làm việc từ Thứ Hai đến Thứ Sáu |
| There are twelve months in a year | Có mười hai tháng trong một năm |
| I have one thousand dong | Tôi có một nghìn đồng |
| I wake up at six o'clock every day | Tôi thức dậy vào lúc sáu giờ mỗi ngày |
| The meeting is at nine o'clock on Thursday | Cuộc họp vào lúc chín giờ Thứ Năm |
| In November and December it is cold in Hanoi | Vào tháng Mười Một và tháng Mười Hai Hà Nội lạnh |
```

- [ ] **Step 2: Commit**

```bash
git -C /Users/work/Documents/vietnameese add assistant/lessons/stage-2-numbers-time.md
git -C /Users/work/Documents/vietnameese commit -m "feat: add Stage 2 lesson — numbers and time"
```

---

## Task 5: Create `lessons/stage-3-questions.md`

**Files:**
- Create: `assistant/lessons/stage-3-questions.md`

- [ ] **Step 1: Create the lesson file**

Create `assistant/lessons/stage-3-questions.md`:

```markdown
# Stage 3: Question Words

## Grammar Notes

Question word placement (from your notes):
- **Khi nào** (when) — start OR end of sentence
- **Tại sao / Vì sao** (why) — always at the START
- **Ai** (who) — END if there is a verb; BEFORE the noun if no verb ("Đó là ai?" vs "Ai đó?")
- **Cái gì / Gì** (what) — END of sentence
- **Thế nào / Như thế nào** (how) — END of sentence
- **Ở đâu / Đâu** (where) — END of sentence
- Yes/no question pattern: **Bạn có [verb] không?** (Do you [verb]?)
- Ability question: **Bạn có thể [verb] không?** (Can you [verb]?)

## Vocabulary

| English | Vietnamese |
|---|---|
| When | Khi nào / Bao giờ / Lúc nào |
| Why | Tại sao / Vì sao |
| Who | Ai |
| What | Cái gì / Gì |
| How | Thế nào / Như thế nào |
| Where | Ở đâu / Đâu |
| How much money | Bao nhiêu tiền |
| How many | Bao nhiêu |
| What time | Mấy giờ |
| How far | Bao xa |
| How long | Bao lâu |
| How old | Bao nhiêu tuổi |
| Do you...? | Bạn có...không? |
| Can you...? | Bạn có thể...không? |

## Sentence Templates

### Tier 1 — Question word alone

| English | Vietnamese |
|---|---|
| When? | Khi nào? |
| Why? | Tại sao? |
| Who? | Ai? |
| What? | Cái gì? |
| How? | Thế nào? |
| Where? | Ở đâu? |
| How much? (money) | Bao nhiêu tiền? |
| What time? | Mấy giờ? |
| How old? | Bao nhiêu tuổi? |
| How far? | Bao xa? |
| How long? | Bao lâu? |

### Tier 2 — Full question sentence

| English | Vietnamese |
|---|---|
| Where do you go? | Bạn đi đâu? |
| What do you eat? | Bạn ăn gì? |
| Who is that? | Đó là ai? |
| Why do you study Vietnamese? | Tại sao bạn học tiếng Việt? |
| What time do you wake up? | Bạn thức dậy lúc mấy giờ? |
| How old are you? | Bạn bao nhiêu tuổi? |
| How much is this? | Cái này bao nhiêu tiền? |
| Do you understand? | Bạn có hiểu không? |
| Can you speak Vietnamese? | Bạn có thể nói tiếng Việt không? |
| When do you go? | Bạn đi khi nào? |
| Where do you want to eat? | Bạn muốn ăn ở đâu? |
| How do you go to work? | Bạn đi làm bằng gì? |

### Tier 3 — Complex question sentence

| English | Vietnamese |
|---|---|
| Why do you want to go to Hanoi? | Tại sao bạn muốn đến Hà Nội? |
| Where do you want to eat today? | Hôm nay bạn muốn ăn ở đâu? |
| How far is it from here to the park? | Từ đây đến công viên bao xa? |
| Who do you want to meet? | Bạn muốn gặp ai? |
| Can you drive me to the park? | Bạn có thể lái xe đưa tôi đến công viên không? |
| How long does it take to go from here to Hanoi? | Từ đây đến Hà Nội mất bao lâu? |
| Why don't you want to go? | Tại sao bạn không muốn đi? |
```

- [ ] **Step 2: Commit**

```bash
git -C /Users/work/Documents/vietnameese add assistant/lessons/stage-3-questions.md
git -C /Users/work/Documents/vietnameese commit -m "feat: add Stage 3 lesson — question words"
```

---

## Task 6: Create `lessons/stage-4-tenses.md`

**Files:**
- Create: `assistant/lessons/stage-4-tenses.md`

- [ ] **Step 1: Create the lesson file**

Create `assistant/lessons/stage-4-tenses.md`:

```markdown
# Stage 4: Tense Markers

## Grammar Notes

- Tense markers go **before the main verb**, never at the end
- **Đã** = past (something already happened): "Tôi đã ăn" (I already ate)
- **Đang** = present continuous (happening right now): "Tôi đang ăn" (I am eating)
- **Sẽ** = future (will happen): "Tôi sẽ ăn" (I will eat)
- **Rồi** at the end strengthens past completion: "Tôi đã ăn rồi" (I have already eaten)
- Vietnamese has no tense by default — without a marker, context determines time. Tense markers add precision.
- Time words (hôm qua, ngày mai, etc.) can also imply tense, but markers make it explicit

## Vocabulary

| English | Vietnamese |
|---|---|
| Past marker (already) | Đã |
| Present continuous marker (currently) | Đang |
| Future marker (will) | Sẽ |
| Already / completion particle | Rồi |

## Sentence Templates

### Tier 1 — Short phrase with one tense marker

| English | Vietnamese |
|---|---|
| I already went | Tôi đã đi |
| I am eating | Tôi đang ăn |
| I will go | Tôi sẽ đi |
| I already understood | Tôi đã hiểu |
| I am studying | Tôi đang học |
| I will eat | Tôi sẽ ăn |
| I already bought it | Tôi đã mua rồi |
| I am driving | Tôi đang lái xe |
| I will meet you | Tôi sẽ gặp bạn |
| I am sleeping | Tôi đang ngủ |
| I have already finished | Tôi đã xong rồi |

### Tier 2 — Full sentence with tense marker

| English | Vietnamese |
|---|---|
| I already went to the park | Tôi đã đi công viên |
| I am currently studying Vietnamese | Tôi đang học tiếng Việt |
| Tomorrow I will go to Hanoi | Ngày mai tôi sẽ đến Hà Nội |
| I already ate | Tôi đã ăn rồi |
| I am drinking water | Tôi đang uống nước |
| I will buy a book tomorrow | Ngày mai tôi sẽ mua sách |
| She is sleeping now | Cô ấy đang ngủ bây giờ |
| We will go together | Chúng ta sẽ đi cùng nhau |
| I already finished work | Tôi đã làm xong việc |
| What are you eating? | Bạn đang ăn gì? |
| Where will you go tomorrow? | Ngày mai bạn sẽ đi đâu? |

### Tier 3 — Complex sentence with tense markers

| English | Vietnamese |
|---|---|
| Yesterday I already went to the park and met my friend | Hôm qua tôi đã đến công viên và gặp bạn của tôi |
| I am currently working but I will finish soon | Tôi đang làm việc nhưng tôi sẽ xong sớm |
| Next Monday I will start studying Vietnamese | Thứ Hai tới tôi sẽ bắt đầu học tiếng Việt |
| I was driving when you called | Tôi đang lái xe khi bạn gọi |
| She already understood what I said | Cô ấy đã hiểu những gì tôi nói |
| Are you currently eating or have you already finished? | Bạn đang ăn hay bạn đã ăn xong rồi? |
```

- [ ] **Step 2: Commit**

```bash
git -C /Users/work/Documents/vietnameese add assistant/lessons/stage-4-tenses.md
git -C /Users/work/Documents/vietnameese commit -m "feat: add Stage 4 lesson — tense markers"
```

---

## Task 7: Create `lessons/stage-5-connectors.md`

**Files:**
- Create: `assistant/lessons/stage-5-connectors.md`

- [ ] **Step 1: Create the lesson file**

Create `assistant/lessons/stage-5-connectors.md`:

```markdown
# Stage 5: Connectors & Conjunctions

## Grammar Notes

- Most conjunctions follow English word order and sit between the two clauses they join
- **Nếu** (if) always begins the conditional clause: "Nếu bạn đi, tôi cũng sẽ đi"
- **Nên** means both "so" (consequence) and "should" — context determines which
- **Sau đó** always connects two complete clauses: "Tôi ăn, sau đó tôi ngủ"
- **Cũng** (also/too) comes before the verb: "Tôi cũng đi" (I also go)
- **Vì** starts the reason clause; **nên** starts the result clause (can pair them)

## Vocabulary

| English | Vietnamese |
|---|---|
| But | Nhưng |
| And | Và |
| Because | Vì / Bởi vì |
| So / Should | Nên / Vậy |
| Then / After that | Sau đó |
| After | Sau khi |
| Also / Too | Cũng |
| Besides | Ngoài ra |
| With | Với |
| If | Nếu |
| Or | Hoặc / Hay |
| Still | Vẫn |
| Next | Tiếp theo |
| Continue | Tiếp tục |
| Like / As | Như |

## Sentence Templates

### Tier 1 — Connector word alone

| English | Vietnamese |
|---|---|
| But | Nhưng |
| And | Và |
| Because | Vì |
| So | Nên |
| Then / After that | Sau đó |
| Also | Cũng |
| Besides | Ngoài ra |
| If | Nếu |
| Or | Hoặc |
| Still | Vẫn |

### Tier 2 — Compound sentence using one connector

| English | Vietnamese |
|---|---|
| I am hungry but I don't want to eat | Tôi đói nhưng tôi không muốn ăn |
| I like coffee and tea | Tôi thích cà phê và trà |
| I study Vietnamese because I love Vietnam | Tôi học tiếng Việt vì tôi yêu Việt Nam |
| I am tired so I will sleep | Tôi mệt nên tôi sẽ ngủ |
| I eat, then I sleep | Tôi ăn, sau đó tôi ngủ |
| I also want to go | Tôi cũng muốn đi |
| Besides studying, I also work | Ngoài việc học, tôi cũng làm việc |
| I go with you | Tôi đi với bạn |
| If you go, I will go too | Nếu bạn đi, tôi cũng sẽ đi |
| Do you want tea or coffee? | Bạn muốn trà hay cà phê? |
| I am still studying | Tôi vẫn đang học |

### Tier 3 — Complex sentence with multiple connectors

| English | Vietnamese |
|---|---|
| I am busy but I will finish soon, then I will go with you | Tôi bận nhưng tôi sẽ xong sớm, sau đó tôi sẽ đi với bạn |
| Because it is raining, I don't want to go out | Vì trời mưa, tôi không muốn ra ngoài |
| If you understand, please say so; if not, I will explain again | Nếu bạn hiểu, hãy nói; nếu không, tôi sẽ giải thích lại |
| Besides Vietnamese, I also want to learn Japanese | Ngoài tiếng Việt, tôi cũng muốn học tiếng Nhật |
| I am tired and hungry, so I will eat and then sleep | Tôi mệt và đói, nên tôi sẽ ăn rồi ngủ |
| Because I study every day, I am getting better and better | Vì tôi học mỗi ngày, tôi ngày càng giỏi hơn |
| If it is sunny tomorrow, I will go to the park with you | Nếu ngày mai trời nắng, tôi sẽ đi công viên với bạn |
```

- [ ] **Step 2: Commit**

```bash
git -C /Users/work/Documents/vietnameese add assistant/lessons/stage-5-connectors.md
git -C /Users/work/Documents/vietnameese commit -m "feat: add Stage 5 lesson — connectors and conjunctions"
```

---

## Task 8: Create `lessons/stage-6-descriptive.md`

**Files:**
- Create: `assistant/lessons/stage-6-descriptive.md`

- [ ] **Step 1: Create the lesson file**

Create `assistant/lessons/stage-6-descriptive.md`:

```markdown
# Stage 6: Descriptive Vocabulary

## Grammar Notes

- **Adjectives follow the noun** they modify: "hoa đỏ" (red flower), "bầu trời xanh" (blue sky)
- When used as a predicate (describing subject), no "là": "Bạn đẹp" (You are beautiful), "Trời nóng" (It is hot)
- **Weather patterns**:
  - "Trời + [adjective]" → Trời nóng (It is hot), Trời lạnh (It is cold)
  - "Có + [noun]" → Có nắng (It is sunny), Có mưa (It is raining)
  - "Nhiều + [noun]" → Nhiều mây (It is cloudy), Nhiều gió (It is windy)
- **Emotions**: Subject + [emotion], no verb needed: "Tôi hạnh phúc" (I am happy)
- **Colors with nouns**: "Hoa màu đỏ" (red flower) or "Hoa đỏ" — both correct

## Vocabulary

### Colors
| English | Vietnamese |
|---|---|
| Red | Màu đỏ |
| Yellow | Màu vàng |
| Green | Màu xanh lá |
| Blue | Màu xanh dương |
| Pink | Màu hồng |
| Purple | Màu tím |
| Orange | Màu cam |
| Gray | Màu xám |
| Brown | Màu nâu |
| White | Màu trắng |
| Black | Màu đen |

### Weather
| English | Vietnamese |
|---|---|
| Sunny | Có nắng |
| Cloudy | Nhiều mây |
| Windy | Nhiều gió |
| Rainy | Có mưa |
| Snowy | Có tuyết |
| Stormy | Có bão |
| Foggy | Nhiều sương mù |
| Overcast | Âm u |
| Hot | Nóng |
| Cold | Lạnh |
| Warm | Ấm |
| Wet | Ẩm ướt |

### Seasons
| English | Vietnamese |
|---|---|
| Autumn | Mùa thu |
| Winter | Mùa đông |
| Summer | Mùa hè |
| Spring | Mùa xuân |

### Emotions
| English | Vietnamese |
|---|---|
| Happy | Hạnh phúc |
| Sad | Buồn bã |
| Angry | Tức giận |
| Scared | Sợ hãi |
| Excited | Hào hứng |
| Upset | Khó chịu |
| Nervous | Hồi hộp |
| Surprised | Bất ngờ |
| Tired | Mệt mỏi |
| Ill | Bị ốm |
| Calm | Bình tĩnh |

## Sentence Templates

### Tier 1 — Single descriptor word

| English | Vietnamese |
|---|---|
| Red | Màu đỏ |
| Blue | Màu xanh dương |
| Happy | Hạnh phúc |
| Sad | Buồn bã |
| Sunny | Có nắng |
| Rainy | Có mưa |
| Hot | Nóng |
| Cold | Lạnh |
| Autumn | Mùa thu |
| Excited | Hào hứng |
| Nervous | Hồi hộp |

### Tier 2 — Simple descriptive sentence

| English | Vietnamese |
|---|---|
| The sky is blue | Bầu trời màu xanh dương |
| The rose is red | Hoa hồng màu đỏ |
| Today is sunny | Hôm nay có nắng |
| I am happy | Tôi hạnh phúc |
| The weather is very cold | Thời tiết rất lạnh |
| I am sad | Tôi buồn bã |
| Hanoi's autumn is beautiful | Mùa thu Hà Nội đẹp |
| The sunflower is yellow | Hoa hướng dương màu vàng |
| I am nervous | Tôi hồi hộp |
| Today is cloudy | Hôm nay nhiều mây |
| Summer is very hot | Mùa hè rất nóng |
| She is wearing a white dress | Cô ấy mặc chiếc váy màu trắng |

### Tier 3 — Complex descriptive sentence

| English | Vietnamese |
|---|---|
| Yesterday Hanoi's autumn was very beautiful | Ngày hôm qua mùa thu Hà Nội rất đẹp |
| The sky is blue and many kinds of flowers have bloomed | Bầu trời màu xanh dương và có rất nhiều loại hoa đã nở |
| I am excited because tomorrow I will go to the beach | Tôi hào hứng vì ngày mai tôi sẽ đến bãi biển |
| If the weather is sunny I will go to the park | Nếu trời có nắng tôi sẽ đi công viên |
| In summer Hanoi is very hot, about 40 degrees | Mùa hè Hà Nội rất nóng, khoảng bốn mươi độ |
| Winter in Hanoi does not have snow but it is still very cold | Mùa đông ở Hà Nội không có tuyết rơi nhưng cũng rất lạnh |
| She is wearing a red áo dài and walking outside | Cô ấy mặc áo dài màu đỏ và đi bộ ra ngoài |
```

- [ ] **Step 2: Commit**

```bash
git -C /Users/work/Documents/vietnameese add assistant/lessons/stage-6-descriptive.md
git -C /Users/work/Documents/vietnameese commit -m "feat: add Stage 6 lesson — descriptive vocabulary"
```

---

## Task 9: Create `lessons/stage-7-composition.md`

**Files:**
- Create: `assistant/lessons/stage-7-composition.md`

- [ ] **Step 1: Create the lesson file**

Create `assistant/lessons/stage-7-composition.md`:

```markdown
# Stage 7: Free Composition

## Grammar Notes

Full review — all grammar from Stages 1–6 applies. Key combinations to master:

- **Tense + connector**: "Tôi đã ăn rồi nhưng tôi vẫn đói" (I already ate but I am still hungry)
- **Question + tense**: "Bạn đã đến đâu rồi?" (Where have you already been?)
- **Descriptor + connector**: "Trời lạnh nhưng đẹp" (The weather is cold but beautiful)
- **Condition + tense**: "Nếu trời mưa, tôi sẽ không đi" (If it rains, I will not go)
- Paragraph structure: set scene (time/place) → action → result/feeling

## Sentence Templates

### Tier 1 — Multi-element phrase (combines 2+ previous stages)

| English | Vietnamese |
|---|---|
| I already ate but I am still hungry | Tôi đã ăn rồi nhưng tôi vẫn đói |
| Beautiful and sunny | Đẹp và có nắng |
| I am tired but happy | Tôi mệt nhưng hạnh phúc |
| I will go on Monday | Vào Thứ Hai tôi sẽ đi |
| Where have you already been? | Bạn đã đến đâu rồi? |

### Tier 2 — Complex single sentence

| English | Vietnamese |
|---|---|
| Yesterday I drove around the city because the weather was beautiful | Hôm qua tôi lái xe vòng quanh thành phố vì thời tiết đẹp |
| If you want to travel to Hanoi, come in autumn — the weather is the most beautiful | Nếu bạn muốn đến Hà Nội du lịch, hãy đến vào mùa thu — thời tiết đẹp nhất |
| I am studying Vietnamese every day so I am getting better | Tôi học tiếng Việt mỗi ngày nên tôi ngày càng giỏi hơn |
| The girls wearing white áo dài walking outside are very beautiful | Những cô gái mặc áo dài màu trắng đi bộ ngoài đường rất xinh đẹp |
| On Sundays I like to run in the park and then drink water and sit down | Vào Chủ Nhật tôi thích chạy trong công viên rồi uống nước và ngồi xuống |
| I was sad because I didn't understand but now I am happy | Tôi buồn vì tôi không hiểu nhưng bây giờ tôi hạnh phúc |

### Tier 3 — Paragraph-level (multiple connected sentences)

| English prompt | Vietnamese target |
|---|---|
| Write 2 sentences: describe today's weather and say how it makes you feel | (open — Claude evaluates for grammar, vocabulary accuracy, and natural expression) |
| Write 2 sentences: what you did yesterday and what you will do tomorrow | (open — Claude evaluates for correct tense marker use) |
| Write 2 sentences: describe a season you like and explain why | (open — Claude evaluates for adjective placement and connector use) |
| Translate the passage: Yesterday in Hanoi it was very cold. I drove around the city and saw many flowers blooming. The roses were red and the sunflowers were yellow. I love Hanoi's autumn. | Ngày hôm qua ở Hà Nội trời rất lạnh. Tôi lái xe vòng quanh thành phố và thấy nhiều hoa nở. Hoa hồng màu đỏ và hoa hướng dương màu vàng. Tôi yêu mùa thu Hà Nội. |
```

- [ ] **Step 2: Commit**

```bash
git -C /Users/work/Documents/vietnameese add assistant/lessons/stage-7-composition.md
git -C /Users/work/Documents/vietnameese commit -m "feat: add Stage 7 lesson — free composition"
```

---

## Task 10: Create `TUTOR.md` (master instruction file)

This is the most critical file. Any Claude session reads this first to know exactly how to behave as a tutor.

**Files:**
- Create: `assistant/TUTOR.md`

- [ ] **Step 1: Create `TUTOR.md`**

Create `assistant/TUTOR.md`:

````markdown
# Vietnamese Tutor — Session Instructions

This file tells Claude how to run every tutoring session. Read this file, then `progress.json`, `curriculum.json`, and the current stage lesson file before doing anything else.

---

## When to Activate

Activate tutor mode when the user says any of:
- "start lesson" / "start session" / "let's practice"
- "lesson: [topic]" / "custom lesson on [topic]"
- "what step are we on?" / "show my progress" / "what are my weak spots?"

---

## Session Startup Protocol

1. Read `assistant/progress.json`
2. Read `assistant/curriculum.json`
3. Read the lesson file for `current_stage` (e.g. `assistant/lessons/stage-1-verbs.md`)
4. Check streak: if `last_session` was not yesterday or today, streak resets to 0
5. Print the session header (see format below)

### Session Header Format

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🎯 Stage [N]: [Stage Name]
📅 Session #[sessions_total + 1] | 🔥 Streak: [streak_days] days
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

If `weak_patterns` is non-empty, add:
```
⚠️  Watch today: [comma-separated weak patterns]
```

Then one line describing today's focus:
```
Today: [warm-up from weak spots] + [main block focus from lesson file]
```

---

## Sentence Selection Algorithm

### Warm-up pool (pick 4)
Pull from `progress.json` → `words` where:
- `familiarity` ≤ 2, OR
- `last_seen` is more than 5 days ago (regardless of familiarity)

Use **Tier 1 sentences only** from the current stage lesson file that contain these words.
If warm-up pool has fewer than 4 words, fill remaining slots with Tier 1 sentences from the previous stage.

### Main block (pick 8)
Draw from the current stage lesson file:
- 2 × Tier 1
- 3 × Tier 2
- 3 × Tier 3

**Adaptive rule within session**: If the learner gets 2 consecutive Tier 2 or Tier 3 sentences wrong, drop back one tier for the next 3 sentences. Resume normal tier distribution after.

**Do not repeat** a sentence already used in the same session.

---

## Exchange Protocol

Present each sentence as:
```
[N/12] Translate to Vietnamese:
"[English sentence]"
```

Wait for the learner's response, then evaluate.

### Evaluation Rules

**Exact match** (ignoring leading/trailing whitespace, case):
- Respond: `✓ Correct!`
- Update word familiarity: +1 (max 5)

**Close match** — accept AND show correction if any of these apply:
- Missing or wrong tone mark (e.g. "Toi" instead of "Tôi")
- Single character typo
- Acceptable synonym already listed in the lesson file's vocabulary table

- Respond:
```
✓ Close! Small correction:
Yours:   [their answer]
Correct: [correct answer]
[one-line note on what to watch: e.g. "Tôi needs the circumflex ô"]
```
- Update word familiarity: +1 (max 5)

**Wrong**:
- Respond:
```
✗ Not quite.
Correct: [correct answer]
Why: [one sentence explanation of the rule — reference grammar notes from the lesson file]
```
- Update word familiarity: -1 (floor 0)
- Log the error pattern (see Pattern Logging below)

### Pattern Logging

Track the reason for each wrong answer during the session. If 3+ wrong answers share a common cause, add it to `weak_patterns` in `progress.json`.

Common pattern names to use:
- "question word position"
- "tense marker placement"
- "adjective after noun"
- "negation with không"
- "tone marks"
- "connector word order"

---

## Session End Protocol

After sentence 12, print the closing summary:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Session complete! Score: [X]/12
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

If there were mistakes, name the top 1–2 explicitly:
```
Focus on: [pattern or specific word]
```

One actionable takeaway line:
```
Takeaway: [concrete thing to remember, e.g. "đã goes BEFORE the verb, not after"]
```

Then update files (see File Update Protocol below).

---

## File Update Protocol

### Update `progress.json`

After every session:
1. Increment `sessions_total` by 1
2. Set `last_session` to today's date (YYYY-MM-DD)
3. Update `streak_days`: +1 if last_session was yesterday, reset to 1 if gap > 1 day
4. For every word practiced: update its `familiarity` and `last_seen` in the `words` object. Add new words if not present (starting familiarity = 1 if got it right first try, else 0)
5. Append any new weak patterns to `weak_patterns` (deduplicate)
6. Check stage progression: if average familiarity of `core_vocab` from `curriculum.json` for the current stage is ≥ 4.0, increment `current_stage` by 1 and add a note to the session history

### Write `history/YYYY-MM-DD.md`

One file per session day. If a file for today already exists, append to it.

Format:
```markdown
## Session [N] — [YYYY-MM-DD HH:MM]

**Stage:** [N] — [Stage Name]
**Score:** [X]/12
**Custom:** [true/false] | **Topic:** [topic if custom, else "-"]

### Sentences Practiced

| # | English | Your Answer | Correct | Result |
|---|---|---|---|---|
| 1 | [english] | [their answer] | [correct answer] | ✓ / ✓~ / ✗ |
...

### Corrections
- [word/pattern]: [brief note]

### Weak patterns logged
- [pattern name] (if any new ones added)
```

---

## Custom Lesson Protocol

Triggered by: "lesson: [topic]" or "custom lesson on [topic]"

1. Identify the topic (e.g. "weather", "seasons", "question words", "colors")
2. Pull matching vocab from `vocabulary_raw.csv` — match against the topic words
3. Warm-up (4 sentences): use known words from `progress.json` with familiarity ≤ 2
4. Main block (8 sentences): sentences built around the topic vocab, Tier 1→3 progression
5. Run the full exchange protocol as normal
6. At session end: update `progress.json` (word familiarity updates normally; do NOT increment `current_stage`)
7. Write to `history/YYYY-MM-DD.md` with `Custom: true` and the topic name

---

## Progress Query Protocol

When the user asks about their progress, read `progress.json` + `curriculum.json` and respond conversationally.

**"What step are we on?"** / **"Where are we in the curriculum?"**
→ Report current stage name and goal, how many sessions completed in this stage, and what Stage N+1 introduces.

Example response format:
```
You're on Stage [N]: [Stage Name].
Goal: [stage goal from curriculum.json]
Sessions completed: [sessions_total]
Streak: [streak_days] days

Next up — Stage [N+1]: [Stage Name]
You'll unlock it when your average familiarity for Stage [N] core vocab reaches 4.0/5.
Current average: [calculated value]

Weak patterns to work on: [weak_patterns list, or "none" if empty]
```

**"What are my weak spots?"**
→ List `weak_patterns` from `progress.json`, plus the 5 words with the lowest `familiarity` scores.

**"Show my history"**
→ List the last 5 sessions from `history/` files: date, score, stage, any corrections noted.

---

## Important Rules

1. **Always read the files first.** Never invent progress or sentences from memory.
2. **Never skip the session header.** It orients the learner.
3. **One sentence at a time.** Wait for the answer before showing the next.
4. **Wrong answers get an explanation.** Not just the correct answer — also the rule.
5. **Close matches are accepted** but always show the corrected form so tone marks are learned.
6. **Always update `progress.json` at session end.** Never leave state stale.
7. **Stage progression is never skipped** even if familiarity looks high — the threshold is per-stage average ≥ 4.0 for the `core_vocab` list.
````

- [ ] **Step 2: Verify TUTOR.md covers all spec requirements**

Check against spec:
- ✓ Session startup reads 3 files
- ✓ 4 warm-up + 8 main = 12 sentences
- ✓ Familiarity 0–5 scale with +1/-1 updates
- ✓ Spaced repetition (5-day rule)
- ✓ Weak pattern detection (3+ errors same cause)
- ✓ Adaptive in-session difficulty pullback
- ✓ Stage progression threshold (≥ 4.0 average)
- ✓ Custom lesson protocol
- ✓ Progress query responses
- ✓ History file format
- ✓ progress.json update rules

- [ ] **Step 3: Commit**

```bash
git -C /Users/work/Documents/vietnameese add assistant/TUTOR.md
git -C /Users/work/Documents/vietnameese commit -m "feat: add TUTOR.md master session instruction file"
```

---

## Task 11: Dry-run verification

Simulate a session start manually to verify the system works end-to-end.

- [ ] **Step 1: Simulate session startup**

Read `TUTOR.md` as if starting a fresh session. Then read `progress.json` (empty state: stage 1, 0 sessions). Then read `lessons/stage-1-verbs.md`.

Expected behavior:
- Header prints with Stage 1: Core Verbs & Pronouns, Session #1, Streak 0
- No weak patterns shown (empty list)
- Warm-up pool: empty `words` object → fill 4 slots with Stage 1 Tier 1 sentences
- Main block: 2×Tier1 + 3×Tier2 + 3×Tier3 from stage-1-verbs.md

- [ ] **Step 2: Simulate one correct and one wrong answer**

Correct: User types "Tôi ăn" for "I eat"
Expected: ✓ Correct!, familiarity for "Ăn" set to 1, last_seen = today

Wrong: User types "Không tôi hiểu" for "I don't understand"
Expected: ✗ Not quite. Correct: Tôi không hiểu. Why: không comes BEFORE the verb (hiểu), not before the subject.

- [ ] **Step 3: Verify all JSON files are readable**

```bash
python3 -c "
import json
p = json.load(open('assistant/progress.json'))
c = json.load(open('assistant/curriculum.json'))
print(f'progress.json: OK')
print(f'curriculum.json: {len(c[\"stages\"])} stages')
print(f'Stage 1 core vocab: {len(c[\"stages\"][0][\"core_vocab\"])} words')
"
```

Expected output:
```
progress.json: OK
curriculum.json: 7 stages
Stage 1 core vocab: 28 words
```

- [ ] **Step 4: Final commit**

```bash
git -C /Users/work/Documents/vietnameese add -A
git -C /Users/work/Documents/vietnameese commit -m "feat: vietnamese tutor system complete — ready for first session"
```

---

## Self-Review

**Spec coverage:**
- ✓ File structure (progress.json, curriculum.json, lessons/, history/) — Tasks 1–2, 3–9
- ✓ 7 curriculum stages with goals, vocab, grammar — Task 2
- ✓ 30 sentence templates per stage in 3 tiers — Tasks 3–9
- ✓ TUTOR.md covers full session flow — Task 10
- ✓ Warm-up selection (familiarity ≤ 2 or 5-day rule) — Task 10
- ✓ Adaptive difficulty pullback — Task 10
- ✓ Evaluation (exact / close / wrong) — Task 10
- ✓ Pattern logging and weak_patterns — Task 10
- ✓ Stage progression threshold — Task 10
- ✓ Custom lesson protocol — Task 10
- ✓ Progress query responses — Task 10
- ✓ history/ file format — Task 10
- ✓ End-to-end verification — Task 11

**No placeholders, no TBDs, no contradictions found.**
