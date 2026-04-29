---
name: start
version: 2.0.0
description: Start or resume the Claude Code learning curriculum. Handles new user onboarding with quick skill assessment, sets up progress tracking, and guides to current lesson. Use when asked to "start", "begin learning", "resume", "continue learning", "where was I", "start the course", "I finished reading", "ready for quiz", or "quiz me now".
user-invocable: true
---

# Start — Learning Curriculum Entry Point

Entry point for the structured Claude Code curriculum. New users get a quick skill check and personalized starting point. Returning users resume from where they left off.

## Lesson Flow (Feynman-integrated)

```
Read README → /feynman (explain back) → gaps resolved → Quiz (7/10) → Unlock next
```

Status transitions per lesson:
- `unlocked` → user starts → `in_progress`
- `in_progress` → feynman passes → `feynman_passed`
- `feynman_passed` → quiz ≥ 7 → `passed`

## Progress File

Location: `.progress/progress.json`

Schema:
```json
{
  "version": "2.0",
  "user_level": "beginner|intermediate|advanced",
  "started_at": "YYYY-MM-DD",
  "last_active": "YYYY-MM-DD",
  "current_lesson": "01-slash-commands",
  "completed_count": 0,
  "lessons": {
    "01-slash-commands": {
      "status": "unlocked|in_progress|feynman_passed|passed|locked",
      "best_score": null, "attempts": 0, "passed_at": null,
      "feynman": { "status": "none|attempted|passed", "attempts": 0, "last_gaps": [], "passed_at": null }
    },
    "02-memory": {
      "status": "locked", "best_score": null, "attempts": 0, "passed_at": null,
      "feynman": { "status": "none", "attempts": 0, "last_gaps": [], "passed_at": null }
    },
    "03-skills":          { "status": "locked", "best_score": null, "attempts": 0, "passed_at": null },
    "04-subagents":       { "status": "locked", "best_score": null, "attempts": 0, "passed_at": null },
    "05-mcp":             { "status": "locked", "best_score": null, "attempts": 0, "passed_at": null },
    "06-hooks":           { "status": "locked", "best_score": null, "attempts": 0, "passed_at": null },
    "07-plugins":         { "status": "locked", "best_score": null, "attempts": 0, "passed_at": null },
    "08-checkpoints":     { "status": "locked", "best_score": null, "attempts": 0, "passed_at": null },
    "09-advanced-features":{ "status": "locked", "best_score": null, "attempts": 0, "passed_at": null },
    "10-cli":             { "status": "locked", "best_score": null, "attempts": 0, "passed_at": null }
  }
}
```

Lesson order (sequential unlock): 01 → 02 → 03 → 04 → 05 → 06 → 07 → 08 → 09 → 10

Next lesson map:
- 01-slash-commands → 02-memory
- 02-memory → 03-skills
- 03-skills → 04-subagents
- 04-subagents → 05-mcp
- 05-mcp → 06-hooks
- 06-hooks → 07-plugins
- 07-plugins → 08-checkpoints
- 08-checkpoints → 09-advanced-features
- 09-advanced-features → 10-cli
- 10-cli → (complete)

## Instructions

### Step 1: Check Existing Progress

Attempt to read `.progress/progress.json`.

**If file exists and is valid JSON** → Go to Step 1B (Returning User)
**If file does not exist or cannot be parsed** → Go to Step 1A (New User)

---

### Step 1A: New User Setup

Display welcome:

```
# Welcome to Claude Code — Master in a Weekend! 🚀

This curriculum covers 10 lessons: from slash commands to advanced plugins.

How it works:
• Read each lesson at your own pace
• Take a quiz after each lesson (need 7/10 to unlock the next)
• Your progress is saved automatically

First, let's find your starting point with 2 quick questions.
```

**Question 1** (AskUserQuestion, header: "Quick Check — Basics"):
"Which of these have you already done with Claude Code? (select all that apply)"
Options:
1. "Ran `claude` and had a conversation"
2. "Created or edited a CLAUDE.md file"
3. "Used 3+ built-in slash commands (e.g. /help, /compact, /model)"
4. "Created a custom slash command or SKILL.md file"

**Question 2** (AskUserQuestion, header: "Quick Check — Advanced"):
"Which advanced features have you used? (select all that apply)"
Options:
1. "Configured an MCP server (e.g. GitHub, database)"
2. "Set up hooks in settings.json"
3. "Used subagents via .claude/agents/"
4. "Used `claude -p` for scripting or CI/CD"

Count total selections across both questions:
- 0–2 → **Beginner** → starting_lesson = `01-slash-commands`, unlock only lesson 01
- 3–5 → **Intermediate** → starting_lesson = `03-skills`, unlock lessons 01–03
- 6–8 → **Advanced** → starting_lesson = `07-plugins`, unlock lessons 01–07

Create `.progress/` directory if needed, then write `.progress/progress.json` with:
- `user_level`: detected level
- `started_at` and `last_active`: today's date
- `current_lesson`: starting_lesson
- `completed_count`: 0
- All lessons with correct status (unlocked/locked per rules above)

Go to Step 2.

---

### Step 1B: Returning User

Parse progress.json. Update `last_active` to today and save.

Extract: `user_level`, `current_lesson`, `completed_count`, and all lesson statuses.

Build progress bar (10 slots):
- ✅ = status "passed"
- ⚡ = status "feynman_passed" (feynman done, quiz ready)
- 🔵 = status "in_progress" (reading / doing feynman)
- 🔓 = status "unlocked"
- 🔒 = status "locked"

Display:

```
# Welcome back! 👋

Level: [Beginner / Intermediate / Advanced]
Progress: [N]/10 lessons completed

01 [icon] | 02 [icon] | 03 [icon] | 04 [icon] | 05 [icon]
06 [icon] | 07 [icon] | 08 [icon] | 09 [icon] | 10 [icon]

✅ passed  ⚡ quiz ready  🔵 in progress  🔓 unlocked  🔒 locked
```

Use AskUserQuestion (header: "Continue"):
"What would you like to do?"
Options:
1. "Continue — go to my current lesson"
2. "Show full progress dashboard (/progress)"
3. "Jump to a specific unlocked lesson"
4. "Restart from the beginning"

- **Continue** → Go to Step 2 with `current_lesson`
- **Show full progress** → Execute the `/progress` skill behavior inline (see progress SKILL.md)
- **Jump** → AskUserQuestion listing only unlocked/passed lessons as options, set chosen as `current_lesson`, go to Step 2
- **Restart** → Confirm: "This will erase all progress. Are you sure?" (Yes/No). If Yes: delete `.progress/progress.json` content and rewrite as empty, go to Step 1A. If No: go back to this menu.

---

### Step 2: Present Current Lesson

Read `<current_lesson>/README.md`.

Lesson display names:
- 01-slash-commands → "Lesson 1: Slash Commands & Skills"
- 02-memory → "Lesson 2: Memory & CLAUDE.md"
- 03-skills → "Lesson 3: Skills (Deep Dive)"
- 04-subagents → "Lesson 4: Subagents"
- 05-mcp → "Lesson 5: MCP Servers"
- 06-hooks → "Lesson 6: Hooks & Automation"
- 07-plugins → "Lesson 7: Plugins"
- 08-checkpoints → "Lesson 8: Checkpoints"
- 09-advanced-features → "Lesson 9: Advanced Features"
- 10-cli → "Lesson 10: CLI Reference"

From the README, extract:
- First paragraph (overview)
- Learning objectives or key topics (first bulleted list or H2 section)
- Estimated time if present (otherwise estimate: ~20–40 min per lesson)

Check lesson status and feynman fields from progress.json:

Display varies by lesson status:

**[status == "in_progress", feynman.status == "none"]** — not yet started feynman:

```
## 📖 [Lesson Display Name]

[Overview paragraph from README]

What you'll learn:
• [key topic 1 from README]
• [key topic 2]
• [key topic 3]

Estimated reading time: ~[N] minutes

---

Your learning path:
  Step 1: Read  [current_lesson]/README.md
  Step 2: /feynman — explain it back in your own words    ← unlocks quiz
  Step 3: Quiz (7/10 to unlock next lesson)

💡 Feynman technique: teaching is the fastest way to find gaps.
```

**[status == "in_progress", feynman.status == "attempted"]** — feynman tried but not passed:

```
## 📖 [Lesson Display Name] — Feynman in Progress

Feynman attempts: [N]
Gaps from last attempt: [gap1], [gap2]

Gợi ý: đọc lại những phần còn thiếu, rồi chạy /feynman để thử lại.
Quiz sẽ mở khóa sau khi bạn pass Feynman.
```

**[status == "feynman_passed"]** — feynman done, quiz ready:

```
## ⚡ [Lesson Display Name] — Quiz Ready!

✓ Feynman passed ([feynman.passed_at])
Quiz attempts so far: [attempts] [If best_score: | best: [best_score]/10]

Gõ "ready for quiz" hoặc /lesson-quiz để thi ngay.
Cần 7/10 để unlock lesson tiếp theo.
```

Update progress.json: if lesson status is "unlocked", change to "in_progress" and save. Add `feynman` field if missing (backward compat).

---

### Step 3: Handle Triggers After Reading

This skill also triggers on these phrases:

- **"I finished reading" / "done reading" / "finished the lesson"** → Feynman gate check
- **"ready for quiz" / "quiz me now"** → Quiz gate check

#### On "finished reading" / "done reading":

1. Read `.progress/progress.json`, get `current_lesson`
2. If no progress file → "Run `/start` first to set up tracking."
3. Check feynman status:
   - `feynman.status == "passed"` → "Feynman đã xong! Gõ 'ready for quiz' để thi."
   - `feynman.status == "attempted"` → Show gaps, invite `/feynman` retry
   - `feynman.status == "none"` → Prompt:

```
Sẵn sàng bước tiếp theo!

Trước khi quiz, hãy làm Feynman exercise:
giải thích [lesson topic] bằng lời của bạn.

Gõ /feynman để bắt đầu — hoặc thử giải thích ngay bây giờ.
```

#### On "ready for quiz" / "quiz me now":

1. Read `.progress/progress.json`, get `current_lesson`
2. If no progress file → proceed to quiz (standalone mode, no gate)
3. Check lesson status:
   - `"feynman_passed"` or `"passed"` → show quiz intro and proceed
   - `"in_progress"` (feynman not passed) → **block**:

```
## Quiz chưa được mở khóa

Bạn cần hoàn thành Feynman exercise trước.

Feynman status: [none → "chưa thử" / attempted → "đang làm, còn gaps"]
[If attempted]: Gaps: [gap1], [gap2]

Gõ /feynman để tiếp tục.
```

4. If allowed through:

```
## 📝 Quiz: [Lesson Display Name]

Passing score: 7/10 → unlocks next lesson
Feynman: ✓ passed [date]
Attempts so far: [N] [If best: | best: [best_score]/10]
```

Proceed with lesson-quiz for `current_lesson` in gated mode.

## Error Handling

### Progress file corrupted
If JSON parse fails: "Your progress file appears corrupted. Would you like to start fresh? (your lesson content is not affected — only the tracking data will reset)"

### All 10 lessons completed
If `completed_count == 10`, display:

```
## 🎉 Curriculum Complete!

You've mastered all 10 lessons of Claude Code!

[Show score table for all lessons]

What's next:
• Create a plugin for your team
• Set up CI/CD pipelines with `claude -p`
• Build a multi-agent workflow
• Contribute to this guide at [repo]
```

### Lesson README missing
If `<current_lesson>/README.md` not found: warn user and offer to skip to next available lesson.

## Validation

### Should trigger:
- "/start"
- "start the course"
- "begin learning"
- "resume my progress"
- "where was I"
- "continue learning"
- "ready for quiz"
- "I finished the lesson"
- "quiz me now"
- "done reading, quiz time"

### Should NOT trigger:
- "quiz me on hooks" (use /lesson-quiz directly)
- "assess my level" (use /self-assessment)
- "what is a hook" (general question)
- "create a skill for me"
