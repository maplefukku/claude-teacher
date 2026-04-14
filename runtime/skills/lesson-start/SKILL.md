---
name: lesson-start
description: Start a specific lesson within a claude-teacher course. Run from the Teacher Terminal. Loads the lesson definition, creates the lesson's progress file, proposes the first prompt the learner should type in the Work Terminal, and initialises a Claude Code task list for in-session UI. Use when a learner wants to begin or resume a specific lesson.
argument-hint: [course-id] [lesson-id]
allowed-tools: Read, Grep, Glob, Write, Edit
---

# lesson-start

Teacher Terminal only. Starts one lesson. Does not do the lesson.

## Arguments

- `$1` — course id
- `$2` — lesson id

If either is missing, refuse and point at `/course-start`.

## Steps

1. **Load the lesson.** Read:
   - `runtime/courses/$1/manifest.yaml` (confirm lesson `$2` is listed)
   - `runtime/courses/$1/lessons/$2/lesson.md`
   - `runtime/courses/$1/lessons/$2/rubric.md`
   - `runtime/courses/$1/lessons/$2/hints.md`
   - `runtime/courses/$1/lessons/$2/examples.md`
   If any required file is missing, refuse and tell the learner the course
   pack is broken. Do not synthesise missing files.

2. **Enforce prereqs.** For every id in the lesson's `prereqs`, verify it
   appears in `.claude-teacher/state/$1.json`'s `completed_lesson_ids`. If
   not, refuse and list what's missing.

3. **Initialise the lesson progress file.** Write
   `.claude-teacher/progress/$1/$2.json` with this shape (unless it already
   exists, in which case read it and continue where the learner left off):

   ```json
   {
     "course_id": "$1",
     "lesson_id": "$2",
     "status": "in_progress",
     "started_at": "<ISO8601>",
     "last_checkpoint_at": null,
     "attempt_count": 0,
     "hint_level_max": 0,
     "artifacts": [],
     "rubric_scores": {},
     "submission_path": "<from lesson.md frontmatter>",
     "reflection": null,
     "next_recommended_lesson": null
   }
   ```

4. **Update the course state file.** Set `current_lesson_id` to `$2` in
   `.claude-teacher/state/$1.json`.

5. **Present the lesson, briefly.** Do NOT dump the full lesson.md into chat.
   Instead, tell the learner:
   - The objective (1-2 lines from `## Objective`).
   - The completion criteria (bullet list, 3-5 items).
   - Where the full lesson lives on disk so they can read it themselves.
   - The exact submission path.
   - The rubric file path (so they can read what they'll be graded on).

6. **Propose the first Work Terminal prompt.** Look at the lesson type:
   - `reading` → propose "Please open <lesson.md path> in your editor and
     read through it, then open <examples.md> and study the sample."
   - `hands-on` → propose the first concrete command, e.g.
     "In your Work Terminal, run: `mkdir -p scratch/<lesson-id> && cd
     scratch/<lesson-id>`, then open Claude Code there."
   - `project` → propose reading the task statement + pulling any starter
     files.

7. **Create a task list** for in-session UI (the learner will see this as a
   running todo list). Seed it with:
   - `understand-goal`
   - `inspect-lesson-files`
   - `do-the-work`
   - `self-check-against-rubric`
   - `submit`
   - `run /checkpoint`

   Remind the learner this task list is UI only — the real progress lives in
   `.claude-teacher/progress/$1/$2.json`.

8. **Hand off.** Tell the learner: "When you think you're done, run
   `/checkpoint $1 $2` from this Teacher Terminal. If stuck, run
   `/give-hint $1 $2`. Do not ask me to write the solution — I won't."

## Output contract

- Lesson card (objective + completion criteria + submission path).
- First Work Terminal instruction.
- Task list seeded.
- Next command: `/checkpoint` or `/give-hint`.
- One-line reminder about the no-solutions rule.

## Refusals

- Never paste `hints.md` into the chat on lesson start. Hints are revealed by
  `/give-hint`.
- Never paste a full model answer from `examples.md` unless the examples file
  explicitly says it's a safe-to-show reference (most won't).
- Never start a lesson whose prereqs aren't satisfied.
