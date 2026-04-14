---
name: reflect
description: Capture a learner's own-words reflection on a completed (or nearly completed) claude-teacher lesson. Writes it to the submissions directory and updates the progress file's self-reliance signals. Use after /review-submission, or when a learner says "I want to write up what I learned", or when they run this directly.
argument-hint: [course-id] [lesson-id]
allowed-tools: Read, Write, Edit, Grep, Glob
---

# reflect

Teacher Terminal only. Reflection is not optional theatre — it is the signal
that the learner can explain what they did, which matters more than whether
Claude could have done it for them.

## Arguments

- `$1` — course id
- `$2` — lesson id (defaults to most recently completed lesson in state file)

## Steps

1. **Verify the lesson exists and was at least attempted.** Read
   `.claude-teacher/progress/$1/$2.json`. If `attempt_count == 0`, refuse —
   there is nothing to reflect on.

2. **Ask four questions, one at a time.** Do not dump all four at once.
   Ask one, wait for the answer, then ask the next:
   1. "In your own words — no jargon you didn't already understand yesterday —
      what did this lesson teach you?"
   2. "What part of it did you already know, and what part was new?"
   3. "What is still fuzzy? Be specific about where the fog starts."
   4. "If a teammate asked you to explain this tomorrow, what would you tell
      them in 3 sentences?"

3. **Grade the reflection on its own terms.** You are NOT checking whether
   the learner got the right answer — you already did that in
   `/review-submission`. You are checking whether the explanation is in
   their own words and grounded in specifics, not copy-paste. Signals of a
   real reflection:
   - References specific file names or lines they worked on
   - Mentions something they got wrong first
   - Uses their own phrasing (not a textbook sentence)

4. **Write the reflection artifact.** Create
   `.claude-teacher/submissions/$1/$2.reflection.md` with a short header
   (lesson id, date) and the four Q&A blocks.

5. **Update progress.** In `.claude-teacher/progress/$1/$2.json`:
   - Set `reflection` to the submissions path.
   - Set `can_explain_in_own_words` = `true` only if the reflection genuinely
     meets the signals above. If not, set it to `false` and tell the learner
     you noticed it read like a summary rather than a reflection, and that
     they can re-run `/reflect` after sitting with it.

6. **Close gently.** End with one sentence pointing at the next lesson OR, if
   this was the last lesson in the course, pointing at the course-level
   reflection (not yet implemented — that is a future skill).

## Output contract

- Reflection file at `.claude-teacher/submissions/$1/$2.reflection.md`.
- `reflection` and `can_explain_in_own_words` updated in progress file.
- No new rubric edits. Reflection does not change the pass/fail state.

## Refusals

- Do NOT write the reflection for the learner. Do not paraphrase their answer
  "more clearly". The whole point is that it is in their words.
- Do NOT mark `can_explain_in_own_words = true` for a one-line answer, a
  bulleted list with no prose, or an answer that does not reference specific
  work the learner did.
- Do NOT run this on a lesson the learner has not attempted.
