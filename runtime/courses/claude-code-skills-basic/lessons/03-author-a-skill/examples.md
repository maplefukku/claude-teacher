# Examples: pitfall gallery

These are bad `SKILL.md` files. Each one fails the rubric in a
specific, common way. Read them before you write your own draft so
you know what to avoid. None of them should be copied, even after
fixing the obvious problem.

## Pitfall 1: bloated description

The description has become the whole manual. Claude now has to read
a paragraph just to decide whether to route here, and every other
Skill looks cheaper by comparison.

```markdown
---
name: commit-summary
description: This skill helps you write commit messages. First, it runs
  git log to look at the last 10 commits in the repository. Then it
  studies the subject line style and the average length. Then it looks
  at the staged changes and produces a commit message in the same
  style with a subject line and an optional body. Use it whenever you
  need to commit.
allowed-tools: Bash(git log:*), Read
argument-hint:
  - commit range
---

# commit-summary

See description.
```

Why it fails: `description_not_bloated`, `body_has_clear_steps`. The
body is empty because everything was jammed into the frontmatter.

## Pitfall 2: wildcard tools

The author did not want to think about scoping, so they opened the
door all the way.

```markdown
---
name: commit-summary
description: Drafts a commit message from recent git history.
allowed-tools: Bash(*), Read, Write, Edit
argument-hint:
  - range
---

# commit-summary

1. Look at the repo.
2. Run whatever git commands you need.
3. Write a commit message.
```

Why it fails: `allowed_tools_scoped`. Grants `Bash(*)`, plus `Write`
and `Edit` which this Skill has no reason to use. A compromised or
confused invocation could now modify files.

## Pitfall 3: body-only-in-description

The description tries to do double duty as body and routing blurb,
and the body is missing entirely.

```markdown
---
name: commit-summary
description: Run `git log --oneline -n 10`, read the output, then
  write a subject line under 70 characters followed by an optional
  body, matching the style of the recent commits.
allowed-tools: Bash(git log:*), Read
argument-hint:
  - range
---

# commit-summary
```

Why it fails: `description_decision_useful` (it describes *how*, not
*when*), `description_not_bloated`, `body_has_clear_steps`. Claude
has nothing to execute at invocation time.

## Pitfall 4: no argument hint

Looks clean, but users have no idea what they can pass, and Claude
has no hint either.

```markdown
---
name: commit-summary
description: Drafts a commit message grounded in recent git history
  for the current repository.
allowed-tools: Bash(git log:*), Read
---

# commit-summary

1. Run `git log --oneline -n 10` to see recent history.
2. Note the subject line style and typical length.
3. Read the staged changes.
4. Produce a subject line under 70 characters and an optional body.
```

Why it fails: `argument_hint_present`. The rest is acceptable, but
without an `argument-hint` the Skill silently loses the ability to
accept a commit range or scope.

## Pitfall 5: name / directory mismatch

Everything inside the file is fine. The file is in the wrong folder,
or the folder has the wrong name.

```markdown
---
name: commit_summary
description: Drafts a commit message grounded in recent git history
  for the current repository.
allowed-tools: Bash(git log:*), Read
argument-hint:
  - commit range, e.g. HEAD~10..HEAD
---

# commit_summary

1. Run `git log --oneline -n 10`.
2. Read the output, note the style.
3. Draft a subject line under 70 chars plus an optional body.
4. Do not fabricate. If the log is empty, say so.
```

File saved at `.claude/skills/commit-summary/SKILL.md`.

Why it fails: `name_matches_directory`. The directory uses a hyphen,
the `name` field uses an underscore. Claude may fail to load it, or
load it under an unexpected identifier that breaks manual invocation.

## What to notice

Four of these five failures are not about the execution of the Skill
at all. They are about the contract: the frontmatter, the
description, the name, the tool scope. That is the pattern. When a
Skill misbehaves, the instinct is to rewrite the body, but the body
is usually the last thing to fix. Start with the frontmatter, make
sure the description answers "when should I invoke this", make sure
`name` and the directory agree, and make sure `allowed-tools` is
narrow. Only then look at the steps.
