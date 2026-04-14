---
name: give-hint
description: Reveal the next hint level for the current claude-teacher lesson. Hints are a ladder — this skill reveals exactly one rung higher than the learner's current hint_level_max, never skips. Use when the learner is stuck, says "give me a hint", "I'm blocked", or asks for help beyond what the lesson already says.
argument-hint: [course-id] [lesson-id]
allowed-tools: Read, Grep, Glob, Edit
---

# give-hint

Teacher Terminal only. The learner can ask for a hint; they cannot demand the
answer. This skill enforces that.

## Arguments

- `$1` — course id
- `$2` — lesson id (defaults to current lesson from state file)

## Steps

1. **Load progress.** Read `.claude-teacher/progress/$1/$2.json`. If no lesson
   is in progress, refuse and point at `/lesson-start`.

2. **Load the hint ladder.** Read
   `runtime/courses/$1/lessons/$2/hints.md`. Parse the hint levels
   (`### Hint level 1`, `### Hint level 2`, ...). Count how many exist.

3. **Compute the next hint level.** `next = hint_level_max + 1`. If
   `next` exceeds the number of defined hint levels, the learner has exhausted
   hints — refuse and tell them to run `/checkpoint` and talk to the
   teacher-coach about what's actually stuck. Do NOT invent a new hint level.

4. **Gatekeep escalation.** Before revealing level >= 3, the learner must have
   at least 2 `attempt_count` since the last hint, and must have run
   `/checkpoint` at least once on this lesson. If they haven't, refuse softly
   and tell them to try a checkpoint first. (Hint laundering — asking for
   hints instead of attempting — is the main failure mode.)

5. **Reveal exactly one hint.** Print the text of hint level `next`. Do not
   also print levels below it. Do not preview the next level.

6. **Update progress.** Set `hint_level_max = next`. Write back.

7. **Remind.** End with one sentence: "Next, go try this in your Work
   Terminal. If it unlocks, run /checkpoint. If it doesn't, come back and I
   can escalate to hint level {next + 1}."

## Output contract

- Exactly one hint level revealed.
- Progress file updated: `hint_level_max = next`.
- No peeking at future hint levels.
- No full solution at any level unless `hints.md` says so explicitly at the
  final level — and even then the teacher-coach should prefer redirecting to
  `examples.md`.

## Refusals

- Never reveal more than one hint level per invocation.
- Never skip hint levels, even if the learner says "just give me level 4".
- Never reveal the final hint if the learner has not attempted the lesson at
  least `attempt_count >= 2`.
- Never copy content from `examples.md` that the examples file does not mark
  as safe-to-show.
