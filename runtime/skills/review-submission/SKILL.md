---
name: review-submission
description: Formal review and close-out of a claude-teacher lesson. Runs the full rubric, produces a written review for the learner, and — if and only if every rubric item passes — marks the lesson complete in the progress and course-state files. Use when the learner says they want to "submit", "finish the lesson", or "close this out", or when they run this directly.
argument-hint: [course-id] [lesson-id]
allowed-tools: Read, Grep, Glob, Edit, Write
---

# review-submission

Teacher Terminal only. This is the only skill that can mark a lesson
complete. `/checkpoint` is a dry run; this is the final grade.

## Arguments

- `$1` — course id
- `$2` — lesson id

## Steps

1. **Load everything.**
   - `.claude-teacher/progress/$1/$2.json`
   - `.claude-teacher/state/$1.json`
   - `runtime/courses/$1/lessons/$2/lesson.md`
   - `runtime/courses/$1/lessons/$2/rubric.md`
   - `runtime/courses/$1/manifest.yaml`
   - The learner's artifacts at `submission_path` and anywhere else the
     lesson lists.

2. **Delegate full grading to the teacher-coach subagent.** Pass it the
   rubric items, all artifact file contents, and the current progress file.
   Ask it to return the fenced JSON block described in
   `runtime/agents/teacher-coach.md`.

3. **Decide pass/fail.**
   - If any rubric item is `false`, the lesson does NOT advance. Update the
     progress file with the latest scores, write a short review to
     `.claude-teacher/submissions/$1/$2.review.md`, and tell the learner
     which items failed and how to address the most blocking one. Do not
     mark complete.
   - If every rubric item is `true`, proceed.

4. **Write the review artifact.** Create
   `.claude-teacher/submissions/$1/$2.review.md` with:
   - Lesson id + title
   - Completed-at timestamp
   - Rubric table (all items + pass)
   - Self-reliance metrics (`hint_level_max`, `attempt_count`,
     `used_reference_implementation`, `can_explain_in_own_words`)
   - A 3-5 line written review from the teacher-coach (not generic praise —
     reference specific things the learner did)
   - A "what you'll use this for next" line pointing at the next lesson

5. **Mark the lesson complete.**
   - In `.claude-teacher/progress/$1/$2.json`: set `status = "completed"`,
     set `next_recommended_lesson`.
   - In `.claude-teacher/state/$1.json`: append `$2` to
     `completed_lesson_ids`, clear `current_lesson_id`.

6. **Propose the next lesson.** Look at `manifest.yaml` for the next lesson
   whose prereqs are now satisfied. Tell the learner the exact command to
   run: `/lesson-start $1 <next-lesson-id>`.

7. **Offer a reflection.** Tell the learner they can also run `/reflect $1
   $2` now if they want to capture what they learned in their own words —
   this affects the `can_explain_in_own_words` self-reliance signal.

## Output contract

- Review file written at `.claude-teacher/submissions/$1/$2.review.md`.
- Progress and state files updated atomically (or refuse).
- Clear pass/fail message to the learner.
- Next command suggested.

## Refusals

- Do NOT mark complete if any rubric item is failing.
- Do NOT infer rubric passes from "looks good to me" — every item must be
  justified by reading the artifact.
- Do NOT skip the review file. No silent advances.
- If the learner has `hint_level_max` equal to the maximum defined hint level
  AND `used_reference_implementation = true`, still pass them if the rubric
  passes, but say so explicitly in the review so they can self-correct next
  lesson.
