---
name: progress
version: 1.0.0
description: Show the learning progress dashboard for the Claude Code curriculum. Displays completed lessons, quiz scores, unlocked lessons, and overall stats. Use when asked to "show my progress", "progress report", "how far am I", "what have I completed", "my scores", "learning stats", or "/progress".
user-invocable: true
---

# Progress — Learning Dashboard

Reads `.progress/progress.json` and renders a full visual dashboard of curriculum progress: completed lessons, quiz scores, pass rates, and what's unlocked next.

## Instructions

### Step 1: Load Progress

Read `.progress/progress.json`.

**If file does not exist:**
```
No progress found. Start the curriculum with `/start` to begin tracking.
```
End skill.

**If file exists but is corrupted:**
```
Progress file appears corrupted. Run `/start` to diagnose or reset.
```
End skill.

---

### Step 2: Render Dashboard

Parse all fields from progress.json and render the full dashboard.

#### Header stats

```
# 📊 Learning Progress — Claude Code Curriculum

Level: [Beginner / Intermediate / Advanced]
Started: [started_at]          Last active: [last_active]
Completed: [completed_count]/10 lessons   Overall: [completed_count * 10]%
```

#### Progress bar (visual, one line per 5 lessons)

```
Lessons:
  01 [icon] Slash Commands      02 [icon] Memory              03 [icon] Skills
  04 [icon] Subagents           05 [icon] MCP                 06 [icon] Hooks
  07 [icon] Plugins             08 [icon] Checkpoints         09 [icon] Advanced
  10 [icon] CLI

  ✅ passed  🔵 in progress  🔓 unlocked  🔒 locked
```

Icons per status:
- `passed` → ✅
- `feynman_passed` → ⚡ (quiz ready)
- `in_progress` → 🔵
- `unlocked` → 🔓
- `locked` → 🔒

#### Lesson score table

Show only lessons that are not "locked":

```
## Quiz Scores

| Lesson                  | Status       | Feynman      | Best Score | Attempts | Passed On  |
|-------------------------|--------------|--------------|------------|----------|------------|
| 01 Slash Commands       | ✅ passed    | ✓ passed     | 9/10       | 1        | 2026-04-29 |
| 02 Memory               | ⚡ quiz ready| ✓ passed     | —          | 0        | —          |
| 03 Skills               | 🔵 in progress| ✗ 2 gaps    | —          | 0        | —          |
| 04 Subagents            | 🔓 unlocked  | —            | —          | —        | —          |
| ...                     | 🔒 locked    | —            | —          | —        | —          |
```

Feynman column values:
- `feynman.status == "passed"` → `✓ passed`
- `feynman.status == "attempted"` → `✗ [N] gaps` (N = last_gaps.length)
- `feynman.status == "none"` → `—`

For locked lessons, show all columns as "—".

#### Current focus

```
## 📍 Current Lesson

[Lesson display name]

[If status == "in_progress" and feynman.status == "none"]:
  Next step: Read lesson → then /feynman to unlock quiz

[If status == "in_progress" and feynman.status == "attempted"]:
  Feynman in progress ([N] attempts) — gaps: [gap1], [gap2]
  Run /feynman to retry

[If status == "feynman_passed"]:
  ✓ Feynman passed — Quiz ready!
  Run /lesson-quiz or say "ready for quiz"
  [If attempts > 0]: Quiz attempts: [N] | Best score: [best_score]/10

[If status == "passed"]:
  ✓ Completed with [best_score]/10

Next to unlock: [next lesson name]
```

#### Summary stats (only if completed_count > 0)

```
## 📈 Stats

Average quiz score: [mean of all best_scores for passed lessons]/10
Perfect scores (10/10): [count]
Lessons on first try: [count of passed with attempts == 1]
Total quiz attempts: [sum of all attempts]
```

---

### Step 3: Offer Actions

Use AskUserQuestion (header: "What next?"):
Options:
1. "Go to my current lesson"
2. "Take the quiz for current lesson"
3. "Jump to a specific lesson"
4. "Nothing, just checking"

- **Go to current lesson** → trigger `/start` continue flow (load README for current_lesson)
- **Take quiz** → trigger `/lesson-quiz <current_lesson>` in gated mode
- **Jump** → list unlocked/passed lessons, let user pick, then show that lesson's README
- **Nothing** → end skill

## Error Handling

### No lessons passed yet
Show dashboard with all "🔒 locked" except lesson 01, and a motivational note:
"Start your first quiz after reading Lesson 1: `[01-slash-commands/README.md]`"

### Progress is all complete
Show completed dashboard and suggest next steps (team plugin, CI/CD integration, contribution).

## Validation

### Should trigger:
- "/progress"
- "show my progress"
- "progress report"
- "how far am I"
- "what have I completed"
- "show my scores"
- "learning stats"
- "how many lessons done"

### Should NOT trigger:
- "/start" (that's the entry point)
- "what is my level" → use /self-assessment
- "quiz results" for a specific lesson → use /lesson-quiz
