---
name: checkpoint
description: Mid-lesson progress check for claude-teacher. Compares the learner's current artifacts against the lesson rubric, updates the progress file, and tells the learner what's still missing. Does NOT mark the lesson complete — that is /review-submission. Use when the learner says "I think I'm done", "can you check where I am", or when they run this directly.
argument-hint: [course-id] [lesson-id]
allowed-tools: Read, Grep, Glob, Edit, Write
---

# checkpoint

Teacher Terminal only. A checkpoint is a soft evaluation — it never advances
the lesson, it only reports status.

## Arguments

- `$1` — course id
- `$2` — lesson id (defaults to current lesson in `.claude-teacher/state/$1.json`)

## Steps

1. **Load progress.** Read `.claude-teacher/progress/$1/$2.json`. If missing,
   tell the learner to run `/lesson-start` first.

2. **Increment attempt_count.** `attempt_count += 1`, set
   `last_checkpoint_at` to now.

3. **Load the rubric.** Read `runtime/courses/$1/lessons/$2/rubric.md` and
   extract each rubric item (`id`, `pass_signal`, `fail_signal`).

4. **Find the artifacts.** From the lesson's `submission_path`, locate the
   learner's output. Also glob any other paths the lesson.md specifies as
   artifacts. Record them in `artifacts[]` in the progress file.

5. **Delegate grading to the teacher-coach subagent.** Pass it the course id,
   lesson id, rubric items, artifact file contents, and current progress JSON.
   Ask it for:
   - per-rubric pass/fail
   - observed `hint_level_max`
   - `next_recommended_action`
   - explicit `advance_to_next_lesson: false` (this is a checkpoint, never an
     advance)

6. **Merge the coach's decision into the progress file.** Update
   `rubric_scores`. Never lower a rubric score that was previously passing
   unless the underlying artifact actually regressed.

7. **Report to the learner.** Structure:
   - One-line status: `lesson=$2 attempt=<n> passing=<x>/<y>`
   - Table or list of rubric items with current state.
   - "Next step" — exactly ONE actionable thing the learner should do next.
     Not three. One. If there are multiple failures, pick the most blocking
     one.
   - If everything passes, tell the learner to run `/review-submission $1 $2`
     to formally close the lesson.

## Output contract

- Updated progress file on disk.
- Short status line.
- Per-rubric status list.
- Exactly one next action for the learner.

## Refusals

- Do NOT mark the lesson `complete` here. Only `/review-submission` can do
  that.
- Do NOT update `completed_lesson_ids` in the course state file.
- Do NOT reveal hints. If the learner needs help, tell them to run
  `/give-hint`.
- Do NOT rewrite the learner's submission to make it pass. You may quote 1-3
  lines for context, never more.
