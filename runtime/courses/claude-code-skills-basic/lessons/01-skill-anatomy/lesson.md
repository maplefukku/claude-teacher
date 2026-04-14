---
id: l01-skill-anatomy
title: Skill Anatomy
duration_min: 20
type: reading
prereqs: none
module: m1
---

# Lesson 01: Skill Anatomy

## Objective

By the end of this lesson you will be able to read any `SKILL.md` file and explain what each section does and why it exists. You will not write a Skill yet. You are learning to see the parts before you learn to assemble them.

Specifically you should be able to explain, in your own words:

- the `name` field
- the `description` field
- the `argument-hint` field
- the `allowed-tools` field
- the `when-to-use` field
- the body of the Skill (the instructions below the frontmatter)
- the difference between frontmatter and body
- why a Skill directory often contains supporting files alongside `SKILL.md`

## Why this matters

Skills are the unit of reusable behavior in Claude Code. Every Skill you will ever write, share, or debug is a `SKILL.md` file plus some optional supporting files. If you cannot read one fluently, you cannot write one, and you cannot tell why one is misbehaving. This lesson is the "learn the parts of the engine" step before you touch a wrench.

The frontmatter of a Skill is not decoration. It is the contract between your Skill and the Claude Code runtime. The runtime reads the frontmatter to decide when to surface the Skill, what tools the Skill is allowed to touch, and how to present it to Claude. Misreading any one of those fields is a common source of "why won't my Skill trigger" confusion later in the course.

## What you will do

1. Open the sample skill at `/home/user/claude-teacher/runtime/courses/claude-code-skills-basic/lessons/01-skill-anatomy/examples.md`. It contains a complete fictional Skill called `greet-user` presented as a fenced code block.
2. Read the entire file slowly. Read the frontmatter first, then the body. Do not skim.
3. For each of the eight items listed under Objective, form a one- or two-sentence explanation in your head. If you cannot, re-read that part of the sample.
4. Write your explanations to `/home/user/claude-teacher/.claude-teacher/submissions/l01-skill-anatomy.md`. Use one short section per item. Write in your own words — do not copy phrases from the sample skill or from this lesson file. Short and honest beats long and parroted.
5. When you think you are done, run `/checkpoint`. The teacher will read your submission against the rubric and either pass you or point at what is missing.

## Completion criteria

You pass this lesson when all of the following are true:

- The file `/home/user/claude-teacher/.claude-teacher/submissions/l01-skill-anatomy.md` exists.
- It contains a clearly separated explanation for each of the eight items listed under Objective.
- The explanations are in your own words and are specific to what those fields actually do, not generic descriptions of YAML or Markdown.
- You ran `/checkpoint` to submit.

## What you will NOT do yet

- You will not write a new Skill.
- You will not modify the sample skill.
- You will not install anything into your own Claude Code config.
- You will not read other Skills on the internet for reference. The sample in `examples.md` is the only artifact you need for this lesson. Using outside material here will hurt you later because it tends to smuggle in conventions this course has not introduced yet.

Reading first. Building next lesson.
