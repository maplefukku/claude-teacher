# Rubric: Lesson 01 Skill Anatomy

This is the checklist the teacher uses when grading the learner's submission at `/home/user/claude-teacher/.claude-teacher/submissions/l01-skill-anatomy.md`. Each item must pass for the lesson to be marked complete.

## Rubric items

- **id: explains_name_field**
  - Evaluates: Does the learner explain that `name` is the Skill's stable identifier, used by the runtime to refer to the Skill and distinct from the human title?
  - Pass: Learner notes that `name` is an identifier (not a description) and that it must be stable and unique.
  - Fail: Learner says `name` is "just the title" or confuses it with `description`.

- **id: explains_description_field**
  - Evaluates: Does the learner explain that `description` is what Claude sees when deciding whether to invoke the Skill, so it must describe *when* and *why* to use it, not just what it is?
  - Pass: Learner connects `description` to Skill selection / invocation behavior.
  - Fail: Learner treats `description` as generic metadata with no functional role.

- **id: explains_argument_hint**
  - Evaluates: Does the learner explain that `argument-hint` tells Claude (and the user) what arguments the Skill expects, and that it is a hint, not an enforced signature?
  - Pass: Learner identifies both audiences (the model and the human) and notes it is advisory.
  - Fail: Learner calls it a strict parameter list or confuses it with a function signature.

- **id: explains_allowed_tools**
  - Evaluates: Does the learner explain that `allowed-tools` restricts which Claude Code tools the Skill is permitted to invoke, and that this is a safety / scoping mechanism?
  - Pass: Learner identifies the scoping / least-privilege intent.
  - Fail: Learner thinks `allowed-tools` adds tools that did not exist, or ignores the security angle entirely.

- **id: explains_when_to_use**
  - Evaluates: Does the learner explain that `when-to-use` gives the model concrete trigger conditions, complementing `description`, and is read before the body?
  - Pass: Learner describes it as trigger criteria, and distinguishes it from `description`.
  - Fail: Learner treats it as interchangeable with `description` or skips it.

- **id: explains_body_section**
  - Evaluates: Does the learner explain that the body (everything below the frontmatter) is the actual instructions Claude follows once the Skill is invoked?
  - Pass: Learner notes the body is the runtime payload and is only read after the Skill is selected.
  - Fail: Learner conflates body with frontmatter or thinks the body is documentation for humans only.

- **id: distinguishes_frontmatter_vs_body**
  - Evaluates: Does the learner clearly state the split — frontmatter is metadata used for selection and permissions, body is the instructions executed after selection?
  - Pass: Learner can articulate the two phases (selection vs execution).
  - Fail: Learner treats the whole file as one undifferentiated blob.

- **id: identifies_supporting_files_pattern**
  - Evaluates: Does the learner notice that a Skill is typically a directory with `SKILL.md` plus optional supporting files (scripts, templates, references), and explain why that structure is useful?
  - Pass: Learner mentions that `SKILL.md` lives in a directory and can be accompanied by other files the body refers to.
  - Fail: Learner thinks a Skill is only ever a single `SKILL.md` file.

## Self-reliance signals

These are not pass/fail on their own, but they change how the teacher reads the submission. If the learner leaned heavily on hints or copied text, the teacher should push back harder in the checkpoint conversation even if the rubric items technically pass.

- **hint_level_used**: What is the highest hint level the learner requested? Levels 1 and 2 are normal. Level 3 is acceptable for a first Skill lesson. Level 4 means the learner should be asked to re-explain at least two items verbally before passing.
- **attempts**: How many times did the learner run `/checkpoint` before passing? One or two is healthy. Four or more suggests the learner is guessing rather than reading.
- **own_words_vs_copied**: Does the submission reuse phrases verbatim from `examples.md`, `lesson.md`, or `hints.md`? If yes on more than one item, fail the lesson regardless of content accuracy and ask for a rewrite. The point of this lesson is comprehension, and copy-paste does not demonstrate it.
