# Hints: Lesson 01 Skill Anatomy

Four escalating hints for the teacher to offer the learner if they get stuck while reading the sample skill. Always offer the lowest level that is likely to unblock them. Do not skip ahead.

### Hint level 1

Before revealing: The learner has said they are stuck but has not tried to describe any field yet, or has only stared at the file for a minute. Give this nudge first.

Look at the part of the sample between the two `---` lines. That block is the frontmatter. Each key in it is one of the items you need to explain. Read the keys out loud. What do you notice about their order, and which ones look like metadata for the runtime versus instructions for Claude?

### Hint level 2

Before revealing: The learner has attempted at least one or two explanations but is mixing up which field does what. Usually this comes out as "isn't `description` the same as `when-to-use`?" or treating `name` as the title.

Think about the lifecycle. Before Claude ever runs a Skill, something has to decide whether this Skill is the right one for the current user request. What does Claude actually *see* at that moment? It is not the body yet. It is the frontmatter fields. So ask yourself: the `description` field is what Claude sees before invocation — why would that matter for how you should write it? And how is that different from `when-to-use`, which is supposed to give concrete trigger conditions rather than a summary?

### Hint level 3

Before revealing: The learner has the selection-vs-execution idea but is still fuzzy on two or three specific fields, typically `argument-hint`, `allowed-tools`, and the supporting-files pattern.

Partial explanation, enough to unblock:

- `name` is an identifier. It is not a human title. The runtime uses it to refer to this specific Skill, and it has to be unique within your Skills collection. Think of it like a function name.
- `argument-hint` is advisory. It tells Claude (and the human reading the Skill) what kind of input the Skill expects, but nothing enforces it as a strict signature. It is a hint in the literal sense.
- `allowed-tools` is a scoping list. It says "when this Skill is running, Claude may only touch these tools". Its purpose is safety and predictability — a Skill that only needs to read files should not be able to write or run shell commands.
- A Skill is usually a directory, not a single file. `SKILL.md` sits at the root of that directory, and the body of `SKILL.md` can reference other files in the same directory (helper scripts, templates, reference docs). This keeps the main file short and lets the Skill carry its own supporting material.

You still have to write the explanations yourself. Do not copy these sentences. Rewrite them against the actual `greet-user` example so your words are tied to something concrete.

### Hint level 4

Before revealing: This is the last resort. The learner has tried, used hint level 3, and is still stuck on at least one item. Offer this only if they are otherwise going to give up. After revealing this, expect the learner to paraphrase, not copy.

Near-complete walkthrough. The learner still writes the submission in their own words.

- **Frontmatter vs body.** The file has two regions. The frontmatter (between the `---` fences) is metadata the Claude Code runtime reads to decide whether, when, and with what permissions to run this Skill. The body (everything below the closing `---`) is the actual instructions the model follows once the Skill has been selected. Selection happens first, execution second. They are read at different moments for different purposes.
- **`name`.** The stable identifier. Used by the runtime and by other tools to refer to this Skill. Not shown to the end user as a pretty title. Must be unique.
- **`description`.** The one thing Claude sees when deciding whether to invoke this Skill out of many candidates. So it should explain the *purpose and situation*, not just restate the name. Good descriptions answer "when would I reach for this?".
- **`argument-hint`.** A short advisory string showing what the Skill expects as input. It is not a typed signature and nothing rejects a call that ignores it. Its job is to guide Claude's prompt construction and to tell humans reading the Skill what to pass.
- **`allowed-tools`.** A whitelist of Claude Code tools this Skill is permitted to use while it is running. Enforced by the runtime. This is the principle of least privilege: a greeting skill has no business writing files or running shell commands, so it declares only `Read`.
- **`when-to-use`.** Concrete trigger conditions. Where `description` is a summary, `when-to-use` gives situations: "use this when the user greets Claude", "use this when the user asks for a welcome message". This helps Claude route correctly when several Skills look similar.
- **Body.** The instructions. Three or four bullet points in the sample telling Claude exactly what to do once the Skill has been chosen: read the name, format a greeting, avoid certain patterns, and so on. The body is where the Skill's behavior lives. Everything above it exists to make sure the body gets executed at the right time with the right permissions.
- **Supporting files.** The sample is a single `SKILL.md`, but real Skills usually live in a directory and the body can reference sibling files — a template, a lookup table, a helper script. Keeps `SKILL.md` short and lets the Skill ship with its own data.

Now close this hint file and write the submission in your own words. If you paraphrase these bullets without understanding them you will fail the "own words" check on the rubric.
