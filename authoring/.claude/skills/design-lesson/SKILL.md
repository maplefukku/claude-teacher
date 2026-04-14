---
name: design-lesson
description: Deep-dive interview to design a single lesson within a claude-teacher course. Produces the lesson.md body, rubric.md, hints.md (with a graduated hint ladder), and examples.md. Use after /design-course and /design-module, when the author is ready to flesh out a specific lesson.
argument-hint: [course-id] [lesson-id]
allowed-tools: Read, Write, Edit, Glob, Grep
---

# design-lesson

This is where most courses die. Authors rush into lesson writing with vague
objectives and end up with "explain X" lessons that are indistinguishable
from blog posts. Do not let that happen here.

## Arguments

- `$1` — course id
- `$2` — lesson id (must already exist as an entry in the course's
  `modules[*].lesson_ids` — if it doesn't, refuse and point at
  `/design-module`).

## Steps

1. **Load context.** Read `authoring/outputs/$1/course.yaml` and find the
   lesson entry with id `$2`. If not found, refuse. Read the module it
   belongs to so you understand its thematic neighbours.

2. **Run the deep interview.** One question at a time. No batching.

   1. "In one sentence, what is the objective of this lesson? Start with a
      verb the learner will perform."
   2. "What is the ONE thing this lesson teaches? If you list more than one,
      split it into two lessons."
   3. "What is the concrete artifact the learner produces? Give me a file
      path. If there is no artifact, this is a reading lesson — confirm."
   4. "What are the completion criteria? Each one must be checkable by
      reading a file or running a command."
   5. "Where do learners get stuck? List the top 3 confusions you expect.
      These become the hint ladder."
   6. "For each confusion, what is the smallest nudge that would unstick
      them? That is hint level 1. What is the explicit explanation? That is
      hint level 3. What is the near-solution? That is hint level 4. We do
      not write a level 5 unless the lesson is exceptionally hard."
   7. "What would a bad submission look like? Give me 2-3 concrete failure
      modes."
   8. "What should the learner NOT do in this lesson, even though it seems
      related?"

3. **Draft the four files.** Into
   `authoring/outputs/$1/lessons/$2/`:

   - `lesson.md` — start from `authoring/templates/lesson.template.md`, fill
     in frontmatter from the interview answers (id, module_id, title, type,
     duration_min, prereqs, submission_path), then Objective, Why this
     matters, What you will do, Completion criteria, Submission, What you
     will NOT do yet. Keep it under 150 lines.

   - `rubric.md` — start from `authoring/templates/rubric.template.md`. Each
     rubric item must have a pass_signal and fail_signal that a teacher can
     check by reading the artifact. No "looks good to me" rubrics. Include
     the self-reliance signals section.

   - `hints.md` — 3-5 graduated hint levels from the interview answers.
     Level 1 nudge, level 2 pointer, level 3 partial explanation, level 4
     near-solution. Mark each level with `### Hint level N` so `/give-hint`
     can parse them.

   - `examples.md` — sample artifacts or pitfall gallery. If the file
     contains a full reference implementation, add a `safe-to-show: false`
     marker at the top so `/give-hint` and `/lesson-start` know not to paste
     it.

4. **Cross-check.** Read the three files you just wrote and verify:
   - Every completion criterion maps to at least one rubric item.
   - Every rubric item has a pass/fail signal.
   - Every hint level matches a confusion from the interview.
   - Lesson duration_min is consistent with the work described.
   If any check fails, fix it before returning.

5. **Hand off.** Tell the author: "Lesson $2 drafted. Run
   `/design-lesson $1 <next-lesson-id>` to continue, or
   `/compile-course-pack $1` when all lessons are done."

## Output contract

- Four files under `authoring/outputs/$1/lessons/$2/` filled in.
- Cross-check passed.
- No content copied verbatim from the course.yaml — each file stands on its
  own.

## Refusals

- Do NOT write the lesson based on assumptions. If the author skipped a
  question, ask it again.
- Do NOT write a lesson with more than one teaching objective. Split instead.
- Do NOT invent rubric items. Every rubric item must trace to an interview
  answer about completion criteria.
- Do NOT write a reference solution in `examples.md` without the
  `safe-to-show: false` marker.
