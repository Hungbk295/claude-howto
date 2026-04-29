# /feynman — Learn by Teaching

Applies the Feynman Technique to each lesson. You explain the topic in plain language, Claude spots gaps, you fill them. This is the mandatory step between reading a lesson and taking the quiz.

## The Method

> "If you can't explain it simply, you don't understand it well enough." — Richard Feynman

1. **Read** the lesson
2. **Explain** it back as if teaching someone new
3. **Gaps found** → review specific sections
4. **Explain again** until concepts are solid
5. **Quiz unlocked** → test with real questions

## Usage

After reading a lesson:
```
/feynman
```

Or for practice on a specific lesson:
```
/feynman hooks
/feynman 03
```

## In Gated Mode

When `.progress/progress.json` exists, `/feynman` is **required** before the quiz unlocks. Pass threshold: ≥ 60% of core concepts covered with no significant misconceptions.

## In Standalone Mode

Without progress tracking, `/feynman` works as a free learning exercise — no gate, no progress update.

## Related skills

- `/start` — Entry point, guides you through the full flow
- `/lesson-quiz` — Take the quiz (requires feynman pass in gated mode)
- `/progress` — See feynman status per lesson
