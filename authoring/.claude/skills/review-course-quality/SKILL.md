---
name: review-course-quality
description: Use this as a QA pass before shipping a compiled course. Checks hint ladders, rubric signals, lesson length, submission paths, and completion criteria, then files a report. Does not auto-fix.
argument-hint: "[course-id]"
allowed-tools: Read, Write, Grep, Glob
---

# review-course-quality

Audits a course for ship-readiness and emits a QA report. This skill is a
reviewer, not an editor: it never changes course content. It reads the authoring
tree (and the compiled runtime pack, if present) and files issues to
`authoring/outputs/<course-id>/qa-report.md`.

Run this after `/compile-course-pack` but before handing the pack off.

## Preconditions

1. `$1` (course-id) must be provided.
2. `authoring/outputs/<course-id>/course.yaml` must exist.
3. Per-lesson files must live under
   `authoring/outputs/<course-id>/lessons/<lesson-id>/`.

## Checks to perform

For each lesson id referenced in `course.yaml`:

1. Hint ladder depth
   - Read `lessons/<lesson-id>/hints.md`.
   - Count top-level hint entries (lines starting with `## Hint` or `- level:`).
   - Fail if fewer than 3 levels exist.

2. Rubric pass/fail signals
   - Read `lessons/<lesson-id>/rubric.md`.
   - Use `Grep` to confirm each rubric item has both a `pass:` and `fail:`
     signal line. Any rubric item missing either is an issue.

3. Lesson length ceiling
   - Read `lessons/<lesson-id>/lesson.md`.
   - Fail if the file exceeds 500 lines. Suggest splitting into sub-lessons.

4. Submission path present
   - Grep the lesson frontmatter for `submission_path:`. Must be non-empty.

5. Completion criteria present
   - Grep the lesson frontmatter for `completion_criteria:`. Must be a non-empty
     list. A single empty bullet counts as missing.

## Steps

1. Load `course.yaml` and collect every lesson id from every module.
2. For each lesson id, run all five checks above. Record every failure as a
   structured issue: `{lesson_id, check, severity, detail}`.
   - `severity: blocker` for missing submission path, missing completion
     criteria, or fewer than 3 hint levels.
   - `severity: warning` for rubric signals and length ceiling.
3. Also run course-level checks:
   - Every module has a `checkout_question` (blocker if missing).
   - Every `learning_outcome` is referenced by at least one module summary
     (warning if orphaned).
4. Use `Write` to create `authoring/outputs/<course-id>/qa-report.md` with:
   - A header line: `# QA report for <course-id>` and today's date.
   - A `## Summary` section: total lessons, blocker count, warning count, and a
     ship/hold verdict. Ship only if blocker count is zero.
   - A `## Blockers` section listing each blocker grouped by lesson.
   - A `## Warnings` section listing each warning grouped by lesson.
   - A `## Checks performed` section enumerating the five per-lesson checks and
     the two course-level checks, so reviewers know what was NOT checked.
5. Print the verdict and the path to the report. Do not modify any lesson file.

## Refusals

- Do not edit, rewrite, or "fix" any lesson, rubric, or hints file.
- Do not delete or move files.
- Do not touch `runtime/courses/`.

## Output contract

When this skill finishes:
- `authoring/outputs/<course-id>/qa-report.md` exists and contains a summary,
  blockers, warnings, and a checks-performed section.
- The report's verdict line is either `verdict: ship` (blockers == 0) or
  `verdict: hold` (blockers > 0).
- No other file under `authoring/outputs/<course-id>/` has been modified.
- The author has been told the report path and whether to proceed to shipping.
