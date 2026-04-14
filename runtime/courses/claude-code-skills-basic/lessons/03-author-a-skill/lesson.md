---
id: l03-author-a-skill
title: Author a Skill
duration_min: 45
type: hands-on
prereqs: [l02-invoke-and-inspect]
module: m1
---

# Lesson 03: Author a Skill

This is the final lesson of the module. You have inspected existing Skills.
Now you write one from scratch and prove it runs.

## Objective

By the end of this lesson you will have a working Skill at
`.claude/skills/commit-summary/SKILL.md` in your scratch workspace that
Claude can load, manually invoke, and use to draft a commit message from
recent git history.

You are not building a tool that runs git. You are building a Skill that
tells Claude how to run git and what to do with the output.

## The task: build `commit-summary`

Create a Skill called `commit-summary`. When invoked, it should help
Claude look at the last few commits in the current repository and
produce a short summary suitable for a new commit message. The summary
must be grounded in the actual log output, not invented.

Concretely, the Skill should instruct Claude to:

1. Run `git log` with a bounded range (for example the last 10 commits,
   or commits since a base branch) to see recent history and style.
2. Run `git log` against staged or unstaged changes, or read specific
   files, to understand what the new commit is about.
3. Produce a concise commit message: a subject line under ~70
   characters, then an optional body explaining the "why".
4. Match the tone and structure of the recent commits it just read.

The Skill body is the instructions. Claude is the runtime. You are the
author.

## Requirements

Your `SKILL.md` must meet all of these:

- Valid YAML frontmatter that parses. `name` matches the directory name
  exactly (`commit-summary`).
- A lean `description` field that helps Claude decide *when* to invoke
  the Skill. It should mention commit messages and recent git history.
  It should not contain the step-by-step instructions.
- A substantive Markdown body with clear, numbered or bulleted steps
  Claude can follow.
- At least one `argument-hint` entry so the user knows what to pass
  (for example, a commit range or a scope).
- An `allowed-tools` field that is scoped: `Bash(git log:*), Read`. No
  wildcards like `Bash(*)` and no tools you do not need.

## Constraints

- Do not implement git calls yourself in Python, Node, or a shell
  script. The Skill is a Markdown instruction file. Claude does the
  work at invocation time using the allowed tools.
- Do not put the body of the Skill inside the `description` field. The
  description is for routing, the body is for execution.
- Do not grant `Write` or `Edit`. This Skill only reads history and
  produces text in the chat; it does not modify files.

## Steps

Work iteratively. Do not try to nail it on the first draft.

1. **Draft.** Create `.claude/skills/commit-summary/SKILL.md` with
   frontmatter and a first-pass body. Keep the description to one or
   two sentences.
2. **Invoke manually.** In your scratch workspace, ask Claude to run
   the `commit-summary` skill against the current repo. Watch what it
   does. Does it actually run `git log`? Does it read the right range?
3. **Inspect the output.** Is the summary grounded in the log, or did
   Claude guess? If it guessed, your body is not specific enough.
4. **Refine.** Tighten the description if Claude is invoking it at the
   wrong moments. Tighten the body if the output is vague. Remove any
   `allowed-tools` entry you did not end up needing.
5. **Re-invoke.** Run it once more on a repo with real history. Confirm
   the summary is usable.

## Submission

Submit the path to your Skill file:

```
.claude/skills/commit-summary/SKILL.md
```

Along with a short transcript (3-10 lines) showing one successful
manual invocation and the commit message it produced.

## Completion criteria

You are done when:

- The file exists at the path above.
- Frontmatter parses, `name` equals the directory name, and
  `allowed-tools` is scoped to `Bash(git log:*), Read`.
- A manual invocation against a repo with at least 5 commits produces
  a commit message grounded in the real log output.
- You can explain, in one sentence, why the description is written the
  way it is.

## What to do when stuck

- Re-read Lesson 01 for the anatomy of a SKILL.md file.
- Re-read Lesson 02 and run `claude` with one of the built-in skills to
  see a real description and body side by side.
- Only then open `hints.md` in this lesson, and start at level 1. Do
  not jump straight to the highest hint level.
- If Claude will not invoke your Skill, the usual culprit is the
  description, not the body.
