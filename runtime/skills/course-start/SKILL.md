---
name: course-start
description: Start a claude-teacher course. Run this once, in the Teacher Terminal, before the learner begins. Verifies the course exists, checks prerequisites, initialises the progress directory, and proposes the first lesson. Use when a learner says they want to "start a course", "begin learning X", or when they run this command directly.
argument-hint: [course-id]
allowed-tools: Read, Grep, Glob, Write
---

# course-start

You are running in the **Teacher Terminal**. The learner will run a separate
Work Terminal where they actually write code. Do not instruct them to do work
in this terminal.

## Arguments

- `$1` — course id (e.g. `claude-code-skills-basic`).

If `$1` is missing, ask the learner which course they want to start and list
what's available by globbing `runtime/courses/*/manifest.yaml`.

## Steps

1. **Locate the course.** Read `runtime/courses/$1/manifest.yaml`. If it is
   missing, stop and list available courses.

2. **Surface the course card.** From the manifest, read aloud (not verbatim —
   in your own brief summary):
   - Title
   - Audience
   - Goal (bullet list)
   - Non-goals
   - Estimated duration
   - Completion criteria

3. **Check prerequisites.** For each item in `prerequisites`, ask the learner
   to confirm. Do not try to auto-detect — you will guess wrong and it is
   condescending.

4. **Initialise progress state.** Create the directory
   `.claude-teacher/progress/$1/` (use Write on a `.gitkeep` if needed — do
   NOT write per-lesson progress files yet; those are created by
   `/lesson-start`). Also create:
   - `.claude-teacher/logs/`
   - `.claude-teacher/submissions/$1/`
   - `.claude-teacher/state/$1.json` with the shape:
     ```json
     {
       "course_id": "$1",
       "started_at": "<ISO8601>",
       "current_lesson_id": null,
       "completed_lesson_ids": []
     }
     ```
   If `.claude-teacher/state/$1.json` already exists, DO NOT overwrite it —
   tell the learner they already have progress and ask whether to continue or
   restart.

5. **Propose the first lesson.** Identify the first lesson id in
   `manifest.yaml` (the one with no `prereqs` or whose prereqs are all
   satisfied). Tell the learner:
   - The lesson id and title.
   - That they should open a **second Claude Code terminal** in their
     working directory and keep it open as the Work Terminal.
   - That when ready, they run `/lesson-start $1 <lesson-id>` from the
     Teacher Terminal to begin.

6. **Do NOT start the lesson automatically.** Starting is an explicit action.

## Output contract

- A short confirmation that progress is initialised.
- A compact "course card" (5-8 lines).
- The exact command to run next: `/lesson-start $1 <first-lesson-id>`.
- A reminder about the two-terminal model.

## Refusals

- If the learner is already mid-lesson (state file shows a `current_lesson_id`
  that is not in `completed_lesson_ids`), refuse to reinitialise without
  explicit `--restart` confirmation from them.
- If `runtime/courses/$1/` exists but `manifest.yaml` is missing or unparsable,
  refuse and report the course as broken rather than guessing.
