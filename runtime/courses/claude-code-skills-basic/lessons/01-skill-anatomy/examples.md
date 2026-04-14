# Sample Skill: `greet-user`

Below is a complete `SKILL.md` for a small fictional Skill called `greet-user`. Its only job is to produce a friendly greeting when the user introduces themselves by name. It is deliberately simple so that the structure is easy to see. Read the whole thing, including the frontmatter, before looking at the "What to notice" list underneath.

```markdown
---
name: greet-user
description: Produces a short, friendly greeting addressed to a specific person. Use when the user has just told Claude their name, or when the user explicitly asks to be welcomed by name.
argument-hint: <name-of-person-to-greet>
allowed-tools: Read
when-to-use: |
  - The user has just introduced themselves ("Hi, I'm Priya").
  - The user asks Claude to welcome someone by name.
  - A parent task needs a warm one-line opener for a message addressed to a named person.
  Do NOT use this Skill for anonymous greetings, multi-recipient greetings, or formal business salutations.
---

# greet-user

You are generating a single friendly greeting addressed to the person named in the argument.

- Read the provided name exactly as given. Do not shorten, translate, or anglicize it.
- Produce exactly one sentence. It should be warm but not sycophantic, and must include the person's name.
- If a file named `tone.txt` exists in this Skill's directory, read it first and match the tone it describes. Otherwise default to a neutral-friendly tone.
- Never ask the user follow-up questions from inside this Skill. Your only output is the greeting itself.
```

## What to notice

You are not being asked to explain these yet — just notice them as structural features while you read. Your explanations go in the submission file.

- The file has two distinct regions separated by `---` fences.
- The top region is YAML-shaped keys and values.
- One of those keys is a plain identifier string, and another is a longer human-readable sentence.
- One key lists tools, and the list is surprisingly short for a Skill that "does something".
- One key uses a multi-line block to enumerate situations, including a negative ("Do NOT use...").
- The body below the fences is written as direct instructions to Claude, in imperative voice.
- The body refers to another file (`tone.txt`) that is not shown here but is expected to live next to `SKILL.md`. That is a clue about how Skills are packaged on disk.
- The body is short. There is no code, no examples, no long prose — just a handful of bullet rules.
