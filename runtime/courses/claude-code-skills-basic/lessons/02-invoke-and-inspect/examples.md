# Examples: Lesson 02 — Invoke and Inspect

This file is read by the teacher, not (primarily) by the learner. It shows what a good submission looks like *in shape* and lists common mistakes to watch for. It deliberately does not provide a model answer that a learner could copy.

## What a good reflection submission looks like

A passing submission has two sections and roughly this skeleton. The content is the learner's own; only the structure is prescribed.

### "What I observed" section

- A short bullet from Step 1 naming the concrete response Claude gave when `/greet-user alice` was invoked. Specific enough that the teacher can tell the learner actually ran the command.
- A bullet noting whether the slash command appeared in the Work Terminal's slash command list *before* invocation, and what that implied.
- A short bullet from Step 2 describing what was added to the `description` field (Lorem Ipsum, duplicated body, etc.) and roughly how much was added.
- A bullet from Step 2 about whether the Skill still worked after bloating — and, crucially, whether the session felt heavier on turns that did *not* invoke the Skill.

### "Reflection" section

- Answer 1: one short paragraph distinguishing the pre-invocation surface from the post-invocation surface. Both halves of the asymmetry should be named. The learner does not need formal terminology; "always-there vs only-when-called" is enough.
- Answer 2: one short paragraph giving a *reason* for keeping the `description` lean. The reason must reference cost, context, or always-present-ness. A style-only answer is not enough.
- Answer 3: one sentence addressed to a future self. It should read like a rule the learner will actually apply, not a restatement of Answer 2.

A good submission is typically 150 to 300 words in total. Longer is not better here. The test is whether the learner can be specific and brief at the same time.

## Common mistakes

The teacher should be ready to spot these. They are the dominant failure modes on this lesson.

- **"It worked" with no observation.** The learner runs `/greet-user alice`, sees output, and writes a submission that never quotes or paraphrases what Claude actually said. This usually means they executed the step mechanically without watching. Ask them to re-run Step 1 and write down Claude's response *as* they see it.

- **Treating `SKILL.md` as one undifferentiated blob.** The learner writes as though Claude "reads the Skill when called" and never distinguishes frontmatter from body. This is the exact misconception the lesson is designed to break. Issue Hint 1 or Hint 2 and have them re-do the reflection.

- **Correct rule, missing reason.** The learner writes "the `description` should be short" but cannot say *why*. They have pattern-matched to the lesson's conclusion without internalizing it. Issue Hint 3 and ask them to add a sentence about per-turn cost to Answer 2.

- **Bloating the body instead of the description in Step 2.** The learner misreads Step 2 and puts Lorem Ipsum in the body. The Skill still works, nothing feels different, and the learner walks away with the wrong takeaway. If you see a submission that reports "Step 2 felt the same as Step 1", check which field they actually edited — they almost certainly edited the wrong one.
