<!--
  Rubric template. One rubric file per lesson, named rubrics/<lesson-id>.md.
-->

# Rubric: <lesson title>

The teacher checks these items during `/checkpoint` (while the student is
working) and `/review-submission` (when the student believes they are done).
Each item should be independently observable - the teacher must be able to
answer pass or fail without guessing. Keep descriptions concrete and tied to
artifacts in the student's `submission_path`, not to vibes.

## Rubric items

| id | description | pass_signal | fail_signal |
|----|-------------|-------------|-------------|
| <snake_case_id_1> | <what is being checked> | <observable evidence of pass> | <observable evidence of fail> |
| <snake_case_id_2> | <what is being checked> | <observable evidence of pass> | <observable evidence of fail> |
| <snake_case_id_3> | <what is being checked> | <observable evidence of pass> | <observable evidence of fail> |

<!--
  Example filled row (delete before shipping):

  | tests_pass | Unit tests for the parser module run green | `pytest tests/test_parser.py` exits 0 with >0 tests collected | Any test fails, or no tests collected |
-->

## Self-reliance signals

These are meta-signals the teacher tracks alongside the rubric to judge
whether the student earned the completion or leaned too hard on assistance.
They do not gate completion on their own, but they inform how the teacher
coaches the next lesson.

- `hint_level_max`: <TODO - highest hint tier the student reached, e.g. 0-3>
- `attempt_count`: <TODO - number of distinct attempts before passing>
- `used_reference_implementation`: <TODO - true|false; did the student view the worked example?>
- `can_explain_in_own_words`: <TODO - true|false; did the student restate the core idea unprompted during /checkpoint?>
