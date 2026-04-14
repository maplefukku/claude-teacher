---
name: design-module
description: Use this when a course already has course-level goals in course.yaml and the author needs to break it into thematically-focused modules before designing individual lessons.
argument-hint: "[course-id]"
allowed-tools: Read, Edit, Grep, Glob
---

# design-module

Given an existing `authoring/outputs/<course-id>/course.yaml` that has course-level
goals (title, summary, learning_outcomes) but an empty or missing `modules:` array,
this skill interviews the author and populates the module breakdown. It does NOT
design the contents of individual lessons — that belongs to `/design-lesson`.

This skill assumes `/design-course` has already run and produced the course shell.

## Preconditions

1. `$1` (course-id) must be provided. If missing, stop and ask the author.
2. Read `authoring/outputs/<course-id>/course.yaml`. If it does not exist, instruct
   the author to run `/design-course` first and stop.
3. Read `authoring/templates/course.yaml` to confirm the canonical `modules:` shape.

## Steps

1. Summarise the existing course goals back to the author in 3-5 bullets so they
   can confirm the frame before module decomposition.
2. Propose an initial module count (typically 3-6) derived from the
   `learning_outcomes` list. Each outcome should map to at least one module.
3. Interview the author, one question at a time, to converge on module boundaries.
   Required questions:
   - "What is the single thematic focus of this module? (one sentence)"
   - "What does a learner need to already know entering it?"
   - "What is the check-out question that proves they exited the module?"
   - "Roughly how many lessons belong here? (1-6)"
4. Enforce invariants before writing:
   - Each module has exactly one thematic focus. If the author gives two, split it.
   - Each module has a non-empty `checkout_question`.
   - Module ids are kebab-case and unique within the course.
   - `lesson_ids` is a placeholder list of stub ids (e.g. `mod-1-lesson-1`) — do
     not invent lesson titles or content.
5. Use `Edit` to update `authoring/outputs/<course-id>/course.yaml` in place,
   replacing or filling the `modules:` array. Preserve all other top-level keys
   (`id`, `title`, `summary`, `learning_outcomes`, etc.) exactly.
6. Print a short summary table: module id, title, focus, lesson-slot count.
7. Tell the author the next step is `/design-lesson <course-id> <lesson-id>` for
   each stub lesson id.

## Module entry shape (must match template)

```yaml
modules:
  - id: kebab-case-id
    title: Human Readable Title
    summary: One or two sentences of thematic focus.
    checkout_question: A single question that gates module completion.
    lesson_ids:
      - mod-1-lesson-1
      - mod-1-lesson-2
```

## Refusals

- Do not write lesson bodies, rubrics, or hints.
- Do not touch files under `runtime/`.
- Do not create new files — only edit `course.yaml`.

## Output contract

When this skill finishes:
- `authoring/outputs/<course-id>/course.yaml` has a populated `modules:` array
  where every entry has `id`, `title`, `summary`, `checkout_question`, and a
  non-empty `lesson_ids` list of placeholder ids.
- Every `learning_outcome` in the course is covered by at least one module.
- No lesson content has been authored.
- The author has been shown a summary table and told the next command to run.
