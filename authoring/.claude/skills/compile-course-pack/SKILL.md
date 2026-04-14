---
name: compile-course-pack
description: Use this when a course under authoring/outputs is fully designed and needs to be compiled into a runnable pack under runtime/courses. Transforms DSL into the runtime layout without inventing content.
argument-hint: "[course-id]"
allowed-tools: Read, Write, Edit, Bash(mkdir:*), Bash(cp:*), Glob, Grep
---

# compile-course-pack

Takes a fully-designed course in `authoring/outputs/<course-id>/` and compiles it
into a runnable course pack at `runtime/courses/<course-id>/`. This is a pure
transformation step: validate, copy, and emit a manifest. It MUST NOT author new
content, guess missing fields, or silently skip broken lessons.

This skill runs after `/design-course`, `/design-module`, and `/design-lesson`
have completed.

## Preconditions

1. `$1` (course-id) must be provided.
2. `authoring/outputs/<course-id>/course.yaml` must exist.
3. For every lesson id referenced from any module, a directory
   `authoring/outputs/<course-id>/lessons/<lesson-id>/` must exist and contain:
   - `lesson.md`  (body + frontmatter)
   - `rubric.md`
   - `hints.md`
   - `examples.md`

## Steps

1. Read `authoring/outputs/<course-id>/course.yaml`. Parse top-level keys and the
   `modules:` array. Collect the full ordered list of lesson ids.
2. Validate the DSL:
   - Every module has `id`, `title`, `summary`, `checkout_question`, `lesson_ids`.
   - Every lesson id is unique across the whole course.
   - For each lesson id, use `Glob` to confirm the four required files exist.
   - Read each `lesson.md` and confirm its YAML frontmatter declares at least:
     `id`, `title`, `submission_path`, `completion_criteria`.
   - If any check fails, STOP with a loud error that lists every missing piece.
     Do not create partial output.
3. Use `Bash(mkdir:*)` to create:
   - `runtime/courses/<course-id>/`
   - `runtime/courses/<course-id>/lessons/`
   - `runtime/courses/<course-id>/lessons/<lesson-id>/` for each lesson id
4. For each lesson id, use `Bash(cp:*)` to copy the four files from
   `authoring/outputs/<course-id>/lessons/<lesson-id>/` into
   `runtime/courses/<course-id>/lessons/<lesson-id>/`. File names stay the same:
   `lesson.md`, `rubric.md`, `hints.md`, `examples.md`.
5. Use `Write` to emit `runtime/courses/<course-id>/manifest.yaml` summarising:
   - `course_id`, `title`, `summary` (copied verbatim from course.yaml)
   - `compiled_at` (today's date, YYYY-MM-DD)
   - `modules:` list, each with `id`, `title`, and ordered `lesson_ids`
   - `lesson_count` and `module_count`
6. Use `Write` to emit `runtime/courses/<course-id>/INSTALL.md` with a short
   install note: how to point the runtime at this pack and a one-line checksum
   note ("manifest.yaml is the source of truth for the runtime loader").
7. Print a compact tree of what was written and the total lesson count.

## Transformation rules

- NEVER invent missing lessons, hints, rubric items, or frontmatter fields.
- NEVER edit files inside `authoring/outputs/` — this skill is read-only there.
- If a file exists but is empty, treat it as missing and fail.
- If `runtime/courses/<course-id>/` already exists, do not delete it; write into
  it idempotently and warn the author that stale files may remain.

## Output contract

When this skill finishes successfully:
- `runtime/courses/<course-id>/manifest.yaml` exists and matches course.yaml.
- `runtime/courses/<course-id>/INSTALL.md` exists.
- For every lesson id in every module, four files exist under
  `runtime/courses/<course-id>/lessons/<lesson-id>/`.
- No file under `authoring/outputs/` has been modified.
- On failure: nothing new under `runtime/courses/<course-id>/` beyond what was
  already there, and a clear error listing every missing requirement.
