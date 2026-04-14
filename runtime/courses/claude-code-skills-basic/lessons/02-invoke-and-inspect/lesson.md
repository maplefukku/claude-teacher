---
id: l02-invoke-and-inspect
title: Invoke and Inspect
duration_min: 25
type: hands-on
prereqs: [l01-skill-anatomy]
module: m1
---

# Lesson 02: Invoke and Inspect

## Objective

By the end of this lesson you will have installed a real Skill into a real Claude Code session, invoked it, watched what Claude actually loads, and be able to explain — in writing — the difference between:

- (a) the `description` field that Claude sees *before* the Skill is invoked, and
- (b) the body of the `SKILL.md` file that Claude loads *after* the Skill is invoked.

You will also be able to explain why the `description` should stay short and the body can afford to be heavy. This is the single most misunderstood thing about Skills, and learning it by observation now will save you from a category of mistakes later.

You are still not authoring a new Skill. You are running one and watching it.

## Why this matters

Before a Skill is invoked, Claude knows only what the frontmatter advertises. The `description` is part of the prompt Claude is given at all times in that session — it is a standing cost. A 900-word `description` is 900 words of context burned on every single turn, regardless of whether the Skill is ever used. The body of the Skill, on the other hand, is only pulled into context at the moment of invocation. That asymmetry is the entire reason the two fields exist as separate things. You need to see that asymmetry with your own eyes.

## Setup steps

1. Create a scratch working directory somewhere outside this repo. For example:

   ```
   mkdir -p ~/scratch/skills-lesson-02
   cd ~/scratch/skills-lesson-02
   ```

2. Inside the scratch directory, create the Skill folder layout that Claude Code looks for:

   ```
   mkdir -p .claude/skills/greet-user
   ```

3. Copy the sample Skill from Lesson 01 into this new location. You want the contents of the fenced `SKILL.md` block in `lessons/01-skill-anatomy/examples.md` to live at:

   ```
   .claude/skills/greet-user/SKILL.md
   ```

   Copy only the body of the fenced code block. Do not include the triple backticks.

4. Open a **Work Terminal** — a fresh Claude Code session — with its working directory set to the scratch directory you just created. This session must be separate from the one running `claude-teacher`. You want it completely uncontaminated so you can observe the Skill in isolation.

## Steps to perform

### Step 1 — invoke the Skill as-is

In the Work Terminal, run:

```
/greet-user alice
```

Observe:

- Did Claude Code pick up the Skill at all? (If `/greet-user` is not offered as a slash command, your directory layout is wrong — fix it before moving on.)
- What did Claude say back?
- Before you typed the slash command, did Claude appear to "know about" the Skill? What cue told you so? (Hint: look at the slash command list, not at Claude's prose.)
- After you typed the slash command, did the response read as though Claude had new, more detailed instructions than it had a second ago?

Take short notes as you go. You will need them for the submission.

### Step 2 — bloat the description and re-invoke

Now edit `.claude/skills/greet-user/SKILL.md` and make the `description` field absurdly long. Aim for at least several paragraphs of filler — copy a chunk of Lorem Ipsum in, or paste the body of the Skill into the description field as well. The goal is to make the `description` visibly heavier than it has any business being. Do **not** change the body of the Skill.

Save the file, start a **new** Work Terminal session in the same scratch directory (so the frontmatter is re-read fresh), and run:

```
/greet-user alice
```

Observe:

- Does the Skill still work? (It should.)
- Does the session feel different before you even invoke the Skill? In particular: are you paying a context cost just for the Skill existing, even on turns where you never call it?
- What does this tell you about where "heavy content" belongs?

Again, take notes.

## Reflection questions

Answer all three in your submission file, in your own words:

1. What is the concrete difference between what Claude sees from a Skill *before* invocation and what Claude sees from a Skill *after* invocation? Describe it as though you were explaining it to someone who just finished Lesson 01.
2. Why is it a bad idea to put detailed instructions, examples, or long prose into the `description` field — even though it would "work" in the sense that Claude would still read them?
3. Given what you observed in Step 2, write one sentence you could tell a future version of yourself about how to decide what goes in the `description` versus what goes in the body.

## Submission

Write your notes and answers to:

```
/home/user/claude-teacher/.claude-teacher/submissions/l02-invoke-and-inspect.md
```

Structure:

- A short "What I observed" section with bullet notes from Step 1 and Step 2.
- A "Reflection" section with the three numbered answers.

Then run `/checkpoint`. The teacher will read your submission against the rubric and either pass you or tell you which observation you did not actually make.

## Completion criteria

You pass this lesson when all of the following are true:

- You installed the sample Skill under a scratch `.claude/skills/greet-user/SKILL.md`.
- You successfully invoked `/greet-user alice` from a Work Terminal rooted in that scratch directory.
- You observed the Skill working a second time after bloating the `description`.
- Your submission file exists at the path above and answers all three reflection questions in your own words.
- Your answers explicitly acknowledge the context-cost difference between `description` and body.
- You ran `/checkpoint`.

## What you will NOT do yet

- You will not edit the body of the sample Skill.
- You will not write a new Skill from scratch.
- You will not install the Skill into your global `~/.claude/skills/` directory. The scratch layout is deliberate — it keeps this lesson isolated and disposable.
