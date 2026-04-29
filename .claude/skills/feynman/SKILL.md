---
name: feynman
version: 1.0.0
description: Feynman Technique learning session for a Claude Code lesson. User explains concepts in their own words, Claude identifies gaps and guides simplification through iteration. Mandatory step before quiz in gated mode. Use when asked to "feynman", "explain back", "I want to practice explaining", "teach it back", "feynman exercise", or when user finishes reading a lesson.
user-invocable: true
---

# Feynman — Learn by Teaching

Applies the Feynman Technique to each lesson: you explain the topic in plain language, Claude spots what's missing or unclear, then guides you to fill the gaps. In gated mode, this must pass before the quiz unlocks.

## The Feynman Technique (4 steps)
1. **Concept** — Identify what you need to understand
2. **Teach** — Explain it as if to someone who knows nothing
3. **Gap** — Find where your explanation breaks down
4. **Simplify** — Return to source, then explain again more clearly

## Progress Schema (feynman fields added to each lesson)

```json
{
  "lesson-id": {
    "status": "unlocked|in_progress|feynman_passed|passed|locked",
    "feynman": {
      "status": "none|attempted|passed",
      "attempts": 0,
      "last_gaps": [],
      "passed_at": null
    }
  }
}
```

## Instructions

### Step 1: Determine Context

Attempt to read `.progress/progress.json`.

**If file does not exist** → **Standalone mode**: ask which lesson to practice:
Use AskUserQuestion (same lesson list as lesson-quiz Step 1). Set `lesson = chosen`. No progress updates. Skip to Step 2.

**If file exists** → **Gated mode**: load `current_lesson`.

Check lesson status:
- `"locked"` → "This lesson is locked. Run `/start` to see your current lesson." End.
- `"passed"` → Practice mode: "You've already passed this lesson. Running Feynman as a refresher — no progress will be updated." Set mode = practice. Continue.
- `"in_progress"` or `"feynman_pending"` → Gated mode. Continue.
- `"feynman_passed"` → "You've already passed the Feynman step! Ready to quiz? Say 'ready for quiz' or run `/lesson-quiz`." End (or offer to redo as practice).

Also check `feynman.attempts` — if ≥ 3 and feynman.status is still "attempted", activate **guided mode** (see Step 3B).

---

### Step 2: Extract Core Concepts (silent)

Read `<lesson_directory>/README.md`.

Internally extract **4–6 core concepts** from the README. Do NOT show this list to the user yet — it's your evaluation rubric.

**What counts as a core concept:**
- A key term or mechanism explained in the README
- A distinction or comparison (e.g., "slash commands vs. skills", "HTTP vs stdio transport")
- A workflow or process the user should understand
- A "when to use" rule

Keep each concept to one sentence. Example for `01-slash-commands`:
1. Custom skills are defined in SKILL.md files with YAML frontmatter
2. `$ARGUMENTS` passes user input dynamically into a skill
3. `disable-model-invocation: true` prevents auto-triggering; `user-invocable: false` hides from menu
4. Skills load in 3 progressive levels to keep context budget low
5. Skills in `.claude/skills/` override legacy `.claude/commands/` files

---

### Step 3A: Standard Feynman Prompt

Display:

```
## Feynman Exercise: [Lesson Display Name]

Bước 1 xong — bạn đã đọc lesson.
Bây giờ: giải thích [lesson topic] như thể bạn đang dạy cho
một người chưa biết gì về Claude Code.

Không cần hoàn hảo. Dùng lời của bạn. Dùng ví dụ nếu giúp ích.
Tránh copy-paste từ tài liệu — paraphrase bằng ngôn ngữ tự nhiên nhất.

Hãy trả lời:
  1. Nó là gì?
  2. Hoạt động như thế nào?
  3. Khi nào dùng?
  4. Một ví dụ cụ thể bạn nhớ
```

Wait for user's free-form explanation (conversational turn, no AskUserQuestion).

---

### Step 3B: Guided Mode (after 3+ failed attempts)

Instead of the standard prompt, Claude explains first:

```
## Guided Feynman: [Lesson Display Name]

Phần này có vẻ khó nắm. Để tôi giải thích theo cách Feynman trước,
rồi bạn thử diễn đạt lại bằng lời của mình.

[Claude explains each gap concept from feynman.last_gaps in plain language,
 with analogy or concrete example. No jargon.]

---
Bây giờ thử bạn: dựa vào những gì bạn vừa đọc, giải thích lại
[gap areas] như thể bạn đang giải thích cho người khác.
```

Wait for user response. Proceed to Step 4.

---

### Step 4: Evaluate Explanation

Evaluate the user's explanation against your internal list of core concepts using 3 dimensions:

**1. Coverage** — Which concepts did the user address? (explicit or implicit)
Mark each concept: ✓ covered / △ partial / ✗ missing

**2. Accuracy** — Any misconceptions or wrong statements?
Flag if significant (would cause real-world errors). Ignore minor imprecision.

**3. Simplicity** — Did they use plain language? Or just repeat jargon without explanation?
If they used terms without explaining them, count as △ partial.

**Pass threshold:**
- Coverage: ≥ 60% of concepts are ✓ or △ (at least partially addressed)
- Accuracy: No significant misconceptions
- If misconception exists → always fail regardless of coverage

---

### Step 5: Present Results

#### On PASS

```
## Feynman Check ✓

Bạn đã nắm được những điểm cốt lõi:

✓ [concept 1] — [one-line affirmation of what they said correctly]
✓ [concept 2] — [...]
△ [concept N] — [close, could be sharper: ...]

[If any partial △]:
Những điểm này đúng hướng nhưng có thể làm rõ hơn. Không blocking.

---
Quiz đã được mở khóa 🎉
Gõ "ready for quiz" hoặc `/lesson-quiz` khi bạn sẵn sàng.
```

Update progress.json (gated mode only):
- `lesson.status` → `"feynman_passed"`
- `lesson.feynman.status` → `"passed"`
- `lesson.feynman.attempts` → increment by 1
- `lesson.feynman.passed_at` → today's date
- `lesson.feynman.last_gaps` → `[]`
- Save file

#### On FAIL (gaps found)

```
## Feynman Check — Gaps Found

Bạn đã nắm được:
✓ [concepts covered]

Cần làm rõ thêm:

✗ [Gap concept 1]
  Vấn đề: [what was missing or wrong — 1-2 sentences plain language]
  Gợi ý: đọc lại phần "[specific section name]" trong README

✗ [Gap concept 2]
  Vấn đề: [...]
  Gợi ý: [specific section]

[If misconception]:
⚠️ [Misconception found]
  Bạn nói "[quote their statement]"
  Thực ra: [correct explanation in plain language]
```

Update progress.json (gated mode only):
- `lesson.feynman.status` → `"attempted"` (do not change lesson.status)
- `lesson.feynman.attempts` → increment by 1
- `lesson.feynman.last_gaps` → [array of gap concept strings]
- Save file

Then offer AskUserQuestion (header: "Tiếp theo"):
"Bạn muốn làm gì?"
Options:
1. "Giải thích lại — tôi đã hiểu thêm rồi"
2. "Cho tôi xem giải thích mẫu cho phần tôi thiếu"
3. "Đọc lại README rồi quay lại sau"

- **Giải thích lại** → Go back to Step 3A, but only ask user to address the gap concepts (not full re-explain)
- **Cho xem mẫu** → Go to Step 5B (show model explanation)
- **Đọc lại** → Display reminder of gaps, end skill

---

### Step 5B: Model Explanation for Gaps

For each gap concept, Claude explains it using Feynman style:
- Plain language, no jargon
- Use analogy if helpful (compare to something familiar)
- Give one concrete example

```
## Explanation: [Gap concept]

[Plain language explanation]

Analogy: [Simple analogy if applicable]

Example: [Concrete 1-sentence example]
```

After explaining all gaps:

```
---
Bây giờ thử lại: giải thích lại [gap concepts] bằng lời của bạn.
```

Go back to Step 3A (focused re-explain).

---

## Error Handling

### README not found
"Không tìm thấy README cho lesson này. Kiểm tra lại cấu trúc thư mục."

### User explanation is very short (< 2 sentences)
Gently prompt: "Thú vị! Bạn có thể nói thêm không? Cụ thể là [one of the core concepts] — bạn hiểu nó như thế nào?"

### User asks "what should I explain?"
Respond: "Hãy tưởng tượng bạn gặp một junior dev chưa biết gì về [lesson topic]. Bạn sẽ giải thích [lesson topic] cho họ bắt đầu từ đâu?"

### User says they haven't read the lesson yet
"Hãy đọc [lesson_directory]/README.md trước, rồi quay lại đây. Feynman chỉ hiệu quả sau khi bạn đã tiếp xúc với material."

## Validation

### Should trigger:
- "/feynman"
- "/feynman hooks"
- "feynman exercise"
- "explain back"
- "teach it back"
- "I want to explain what I learned"
- "practice explaining lesson 3"
- "I finished reading"  ← also triggers /start Step 3

### Should NOT trigger:
- "explain hooks to me" (user asking Claude to explain — opposite direction)
- "quiz me on hooks" → use /lesson-quiz
- "assess my level" → use /self-assessment
