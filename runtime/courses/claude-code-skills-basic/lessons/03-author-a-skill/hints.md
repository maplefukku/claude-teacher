# Hints: Author a Skill

Use these in order. Stop at the lowest level that unblocks you. Every
level you skip past is a data point about your current understanding,
so do not jump to the bottom.

### Hint level 1 — orient

Before touching your draft, answer two questions out loud:

- What is the one sentence that would tell Claude "use `commit-summary`
  right now"?
- What does Claude need to *read* before it can write a commit
  message?

If you cannot answer both, re-open Lesson 01 (skill anatomy) and
Lesson 02 (invoke and inspect). The answer to the first question is
your `description`. The answer to the second is the core of your
body.

### Hint level 2 — structure check

Open your draft and verify, in order:

1. There is a `---` line, then frontmatter, then another `---`, then
   the body. No blank line before the first `---`.
2. Frontmatter has `name`, `description`, `allowed-tools`, and at
   least one `argument-hint`.
3. `name` is exactly the directory name.
4. `allowed-tools` contains only what you actually use. For this
   Skill, that is `Bash(git log:*), Read`.

If any of these are off, fix them before invoking.

### Hint level 3 — description triage

If Claude is invoking your Skill at the wrong times, or not at all,
the description is the problem. Try this rewrite exercise:

- Write the description in one sentence that starts with a verb or a
  noun phrase, not with "This skill...".
- Include the words `commit message` and a reference to recent git
  history.
- Cut any sentence that tells Claude *how* to do the work. That
  belongs in the body.

Keep it under ~200 characters. If you cannot, you are trying to make
the description do the body's job.

### Hint level 4 — body triage

If invocation works but the output is vague or invented, your body is
under-specified. Concrete fixes:

- Name the exact git command form you want Claude to run, for example
  `git log --oneline -n 10` or `git log <base>..HEAD`.
- Tell Claude what to do with the output: look at subject line style,
  average length, whether bodies are used, whether scopes are used.
- Tell Claude what the final answer should look like: a subject line
  under ~70 chars, optional body, no invented claims.
- Tell Claude explicitly: if the log is empty or unreadable, say so
  and stop. Do not fabricate.

### Hint level 5 — skeleton with placeholders

This is the highest hint level. It is a skeleton, not a solution. You
still have to write the words. If you copy this verbatim, the rubric
items for description and body will fail.

```markdown
---
name: commit-summary
description: <ONE SENTENCE, under 200 chars. When should Claude reach
  for this skill? Mention commit messages and recent history. Do not
  put instructions here.>
allowed-tools: Bash(git log:*), Read
argument-hint:
  - <e.g. "commit range, like HEAD~10..HEAD">
---

# commit-summary

<ONE SHORT PARAGRAPH describing the intent of this skill in your own
words. Not the description, not the steps.>

## Steps

1. <Tell Claude which git log command(s) to run and why. Be specific
   about the range.>
2. <Tell Claude what to look for in the output: tone, length, scope
   prefixes, body usage.>
3. <Tell Claude how to map the user's current change onto that
   style.>
4. <Tell Claude the exact shape of the final answer: subject line
   length limit, optional body, no fabrication.>

## When not to use

<One or two sentences. This helps Claude avoid invoking at the wrong
moment.>

## Failure modes

- <What to do if the log is empty.>
- <What to do if the user's change is unclear.>
```

Fill in every `<...>` in your own words. The whole point of the
lesson is that *you* make the routing and execution decisions; the
skeleton only arranges the slots.
