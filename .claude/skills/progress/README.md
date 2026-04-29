# /progress — Learning Dashboard

Displays your full learning progress across the 10-lesson Claude Code curriculum.

## Usage

```
/progress
```

Or say: `"show my progress"`, `"progress report"`, `"how far am I"`

## What it shows

- Visual progress bar (✅ passed / 🔵 current / 🔓 unlocked / 🔒 locked)
- Quiz score table with best scores and attempt counts
- Current lesson focus and what's needed to unlock the next
- Summary stats: average score, perfect scores, attempts

## Requirements

Requires `.progress/progress.json` to exist. Run `/start` first to initialize tracking.

## Related skills

- `/start` — Entry point, resumes curriculum
- `/lesson-quiz` — Take a quiz for a specific lesson
