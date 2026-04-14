# Hints: Lesson 02 — Invoke and Inspect

These hints are arranged from vaguest to most explicit. The teacher should hand out the lowest-numbered hint that would unblock the learner — never skip straight to Hint 4.

The focus of all four hints is the same: *why* does the `description` want to be lean while the body can afford to be heavy?

---

### Hint level 1 — nudge

Go back and re-read the `description` field in your `SKILL.md`. Ask yourself: at what moment does Claude actually need to know that text? Is it only after you typed `/greet-user`, or was Claude already carrying that text around before you typed anything?

If you are not sure, try this: start a brand-new Work Terminal, do *not* invoke the Skill, and just chat with Claude for a turn or two about something unrelated. Is the Skill's `description` influencing the session even though you never called it? Sit with that question before moving on.

---

### Hint level 2 — framing

Think of the `description` as a *listing in a menu* and the body as *the recipe you only read once you have ordered the dish*. The menu is always visible to every diner on every visit — it has to be short, because every diner is paying attention to it at all times. The recipe is pulled out of the kitchen drawer only when someone orders it — it can be as long and detailed as the dish demands.

Now re-read your Step 2 observations. When you bloated the `description`, you did not make the recipe heavier — you made the *menu* heavier. Every single turn in that Work Terminal, for the rest of the session, Claude is reading a menu that is dragging around paragraphs of Lorem Ipsum it does not need. That is the cost.

---

### Hint level 3 — direct

Here is the rule, stated plainly. Write it in your own words in the submission — do not copy this phrasing:

- The `description` is in Claude's context **on every turn**, whether or not you ever invoke the Skill.
- The body is in Claude's context **only on the turn when the Skill is invoked** (and typically only briefly, as part of fulfilling that invocation).

So anything you put in the `description` is paid *per turn*, multiplied by *every turn in every session where that Skill is installed*. Anything you put in the body is paid *once, on demand*. A long body is cheap. A long `description` is a tax.

The correct design move is therefore: keep the `description` short enough that Claude can cheaply notice the Skill exists and decide whether to use it, and put everything else — examples, edge cases, step-by-step instructions, domain knowledge — into the body.

---

### Hint level 4 — near-full explanation

If Hint 3 still has not landed, here is the full picture.

A Skill has two surfaces. The frontmatter (and specifically the `description` field within it) is the **advertising** surface. The body is the **payload** surface. They have different lifecycles inside a Claude Code session:

- **Advertising surface (`description`)** — Claude Code injects this into the session prompt at startup and keeps it there. It is how Claude learns that the Skill exists, what it is roughly for, and when to suggest it. Because it is always present, every token in the `description` is a token Claude is re-reading on every user message, even on messages that have nothing to do with the Skill. Bloat here is multiplied by the length of the session.

- **Payload surface (body)** — This is only loaded when the Skill is actually invoked. It can contain long instructions, worked examples, domain-specific rules, edge cases, error handling, and so on, because Claude only pays the cost when the user actually wants the Skill to run. A 2,000-token body that only fires once per session costs 2,000 tokens once. A 2,000-token `description` in a 50-turn session costs roughly 100,000 tokens over the life of the session.

This is why you were asked to observe Step 2 in a *fresh* Work Terminal: you wanted to feel the bloated `description` being paid up front, before you even got to the slash command. That was not a coincidence — it was the entire point of the lesson.

So the rule to carry forward is: **treat the `description` as an index entry, not as documentation.** If you catch yourself wanting to put instructions there, push them down into the body. Your future self will be grateful, and more importantly, every future turn in every future session will be cheaper.
