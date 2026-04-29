# /start — Curriculum Entry Point

The main entry point for the Claude Code learning curriculum.

## Usage

```
/start
```

Or just say: `"start"`, `"begin learning"`, `"resume"`, `"where was I"`

## What it does

**First time:**
1. Runs a 2-question quick skill check (< 1 minute)
2. Detects your level: Beginner / Intermediate / Advanced
3. Sets your starting lesson and unlocks appropriate content
4. Creates `.progress/progress.json` to track your journey

**Returning user:**
1. Shows your progress bar at a glance
2. Lets you continue, jump to an unlocked lesson, or restart
3. Displays your current lesson and quiz history

## Progress gate

Every lesson requires **7/10 on the quiz** to unlock the next one. This ensures you build a solid foundation before advancing.

## After reading a lesson

Say `"ready for quiz"` or `"quiz me now"` — `/start` will pick up from there.

## Related skills

- `/progress` — Full progress dashboard
- `/lesson-quiz` — Take a quiz directly (standalone mode)
- `/self-assessment` — Detailed skill profiling across all 10 areas
