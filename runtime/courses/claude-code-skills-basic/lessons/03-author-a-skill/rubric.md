# Rubric: Author a Skill

Each item is pass/fail. You need all nine to pass the lesson. Signals
below tell you and the grader how to decide.

## Rubric items

### 1. frontmatter_parses
- **Pass:** The YAML block at the top of `SKILL.md` is delimited by
  `---` on both sides and parses as valid YAML. Required keys
  (`name`, `description`) are present.
- **Fail:** Missing delimiters, tab characters inside the block,
  unquoted values containing colons, or a missing required key.

### 2. name_matches_directory
- **Pass:** `name: commit-summary` and the directory is
  `.claude/skills/commit-summary/`.
- **Fail:** Any mismatch, including case or hyphen-vs-underscore
  differences.

### 3. description_decision_useful
- **Pass:** Reading only the description, a reasonable person (or
  Claude) can tell when to invoke this Skill. It mentions commit
  messages and recent git history or similar.
- **Fail:** Description is generic ("a helpful skill"), tool-focused
  ("runs git log"), or refers to the body instead of the trigger.

### 4. description_not_bloated
- **Pass:** Description is under ~200 characters and contains no
  imperative instructions or numbered steps. It is a routing blurb,
  not a manual.
- **Fail:** Description is a paragraph, contains "first do X then do
  Y", or duplicates the body.

### 5. body_has_clear_steps
- **Pass:** The body has an ordered or clearly bulleted sequence of
  steps Claude can follow. Each step is actionable.
- **Fail:** Body is a prose essay with no structure, or the steps
  are vague ("look at the repo and think about it").

### 6. allowed_tools_scoped
- **Pass:** `allowed-tools: Bash(git log:*), Read`. No wildcard,
  no unused tools.
- **Fail:** Uses `Bash(*)`, `Bash`, includes `Write`/`Edit`, or
  omits the field entirely.

### 7. argument_hint_present
- **Pass:** At least one `argument-hint` entry is declared in
  frontmatter and reflects something the user might actually pass
  (e.g. a commit range, a base branch, a scope).
- **Fail:** No argument hint, or a placeholder like `<arg>`.

### 8. manually_invocable
- **Pass:** You can start Claude in the scratch workspace, ask it to
  run the `commit-summary` skill, and it loads and executes without
  errors.
- **Fail:** Claude cannot find the Skill, refuses to load it, or
  errors out on frontmatter parsing.

### 9. produces_reasonable_output_on_sample_repo
- **Pass:** On a repo with at least 5 real commits, invocation
  produces a commit message whose subject and body are clearly
  grounded in the actual `git log` output. You can trace claims in
  the summary back to specific commits.
- **Fail:** Output is generic, invented, or ignores the actual log.

## Self-reliance signals

Record these for yourself. They do not block completion, but they
tell you how this lesson is going.

- **hint_level_max:** The highest hint level you opened. Target: 2
  or lower. Above 3 means you should re-do Lesson 01 before moving
  on.
- **attempts:** How many drafts of `SKILL.md` it took before
  invocation produced usable output. Target: 2-4. One attempt is
  suspicious (did you actually invoke it?); more than 5 suggests you
  are tweaking syntax instead of re-reading the earlier lessons.
- **re_read_prior_lessons:** Did you re-open Lesson 01 or Lesson 02
  before asking for hints? If no and `hint_level_max >= 3`, that is
  the habit to change, not the Skill file.
