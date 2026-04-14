# Rubric: Lesson 02 — Invoke and Inspect

This rubric is read by the teacher when the learner runs `/checkpoint`. Each item is binary: pass or fail. The learner must pass every item to complete the lesson.

## Rubric items

### skill_installed_correctly

- **What is checked:** The learner created a scratch directory containing `.claude/skills/greet-user/SKILL.md` with the sample Skill content from Lesson 01.
- **Pass signal:** The submission describes (or the environment shows) a working file at `<scratch>/.claude/skills/greet-user/SKILL.md`. The learner references the scratch directory path when describing Step 1.
- **Fail signal:** The learner installed the Skill into `~/.claude/skills/`, into the `claude-teacher` repo itself, or never created the directory structure at all. Or they pasted the fenced code block with its backticks still in place.

### skill_invoked_successfully

- **What is checked:** The learner actually ran `/greet-user alice` in a Work Terminal and saw Claude respond as the Skill dictates.
- **Pass signal:** The submission contains a concrete observation of what Claude said back — something that could only have been written by someone who ran the command. Generic "it worked" does not count.
- **Fail signal:** The submission skips Step 1 entirely, or describes Step 1 in abstract terms that could have been produced without running anything.

### observed_description_vs_body_difference

- **What is checked:** The learner can distinguish what Claude knows from the `description` (before invocation) from what Claude loads from the body (after invocation).
- **Pass signal:** Reflection question 1 names both sides of the asymmetry: description is standing / always-present / advertising, body is on-demand / loaded at invocation / heavier content. The language does not have to be precise, but both sides must be present.
- **Fail signal:** The answer conflates the two, or treats the whole `SKILL.md` as a single blob that Claude "reads when called".

### articulated_why_description_is_lightweight

- **What is checked:** The learner can explain *why* the `description` should stay short, not just that it should.
- **Pass signal:** Reflection question 2 connects the shortness of the `description` to a cost paid by Claude on every turn, or to context / prompt budget, or to the `description` always being in view versus the body only being pulled on invocation. Any coherent framing of the asymmetry counts.
- **Fail signal:** The answer says "because it's the description" or "because the body is for that" without explaining the underlying cost. Style-only answers ("it looks cleaner") do not pass.

### identified_context_cost_of_large_descriptions

- **What is checked:** The learner's Step 2 observation noticed that bloating the `description` affects the session even when the Skill is not invoked.
- **Pass signal:** The "What I observed" notes for Step 2 mention a cost, a heaviness, a feeling of wasted context, or in general the point that the bloat was paid *before* the slash command was typed. Reflection question 3 is consistent with this observation.
- **Fail signal:** The learner only noticed that the Skill "still worked" and never registered that the bloat was paid up-front on every turn.

### submission_in_own_words

- **What is checked:** The submission is written by the learner, not copied from the lesson, from the sample Skill, or from a prior submission.
- **Pass signal:** The phrasing is clearly the learner's. Awkward phrasing is fine. Typos are fine. Honesty is fine.
- **Fail signal:** Sentences lifted verbatim from `lesson.md`, from `examples.md`, or from the `description` field of the Skill itself. Unusually polished prose that happens to mirror the lesson file structure.

## Self-reliance signals

These are not graded items — they are prompts the teacher uses to decide whether the learner is genuinely *seeing* the lesson or just completing steps.

- Did the learner take notes during observation, or did they reconstruct the observations after the fact from memory? (Notes are better.)
- When Step 1 failed (for example, wrong directory layout), did the learner debug it themselves by checking the file path, or did they immediately ask the teacher for help?
- In reflection question 3, is the learner speaking to their *future self* in a way that suggests they have internalized the rule, or are they just restating question 2?
- Does the learner ever use the word "advertise" or "preview" or an equivalent to describe the role of `description`? That framing is a strong signal the asymmetry has clicked.
- Did the learner notice anything this lesson did not explicitly point at? (For example: that Skills are loaded per-session, or that the slash command list is itself a lightweight surface.) Unprompted observations are a strong positive signal and should be acknowledged in the teacher's pass message.
