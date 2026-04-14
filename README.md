# claude-teacher

Interactive hands-on LMS that runs inside Claude Code.

`claude-teacher` treats Claude Code not as a chatbot but as a teaching runtime.
Courses are structured data; lessons are skill-launched workflows; the teacher
is a dedicated subagent; progress lives in plain files that survive
compaction, restarts, and terminal switches.

## Why this exists

Most "AI tutors" collapse into "let me do it for you." That is not teaching —
it is delegation. `claude-teacher` enforces a separation of roles so the
learner actually writes code, while Claude Code stays in the teacher seat.

## Repository layout

```
claude-teacher/
├─ authoring/        # Content creation side (for course authors)
│  ├─ .claude/
│  │  └─ skills/     # /design-course, /design-lesson, /compile-course-pack ...
│  └─ templates/     # course.yaml / lesson / rubric templates
│
├─ runtime/          # What learners install
│  ├─ .claude-plugin/plugin.json
│  ├─ skills/        # /course-start /lesson-start /checkpoint /give-hint ...
│  ├─ agents/        # teacher-coach subagent
│  └─ courses/       # Packaged course content (manifest + lessons)
│
├─ learner-workspace/ # Example of what a learner's repo looks like
│  ├─ .claude/settings.json
│  └─ .claude-teacher/
│     ├─ progress/
│     ├─ logs/
│     ├─ submissions/
│     └─ state/
│
└─ docs/             # Architecture and design notes
```

## Two-terminal model

`claude-teacher` is designed to run in two Claude Code terminals at once:

| Terminal         | Role           | Allowed to                                    |
| ---------------- | -------------- | --------------------------------------------- |
| Teacher Terminal | Progress, eval | Read files, read progress, give hints, review |
| Work Terminal    | Actually code  | Edit, run, install, break things              |

The Teacher Terminal is expected to run in plan-style mode (read/analyse
only). The Work Terminal is where the learner types prompts and changes code.

Keeping them separate is the difference between a tutor and a ghost-writer.

## Quickstart (learner side)

1. Install the `runtime/` plugin (see `docs/ARCHITECTURE.md`).
2. In the **Teacher Terminal**, run:
   ```
   /course-start claude-code-skills-basic
   ```
3. Follow the teacher's instructions in the **Work Terminal**.
4. When stuck: `/give-hint`.
5. When ready: `/checkpoint`, then `/review-submission`, then `/reflect`.

## Authoring a new course

In the `authoring/` project:

1. `/design-course` — hearing interview, fills in `course.yaml`.
2. `/design-module` — module-level breakdown.
3. `/design-lesson` — per-lesson deep dive.
4. `/compile-course-pack` — emits a runnable course pack under
   `runtime/courses/<course-id>/`.
5. `/review-course-quality` — internal QA pass before shipping.

## Design principles

- **Skills are launchers, not textbooks.** Long rubrics, examples, hints
  live in supporting files, not in `SKILL.md`.
- **External state is the source of truth.** Task lists are UI; the real
  progress DB is `.claude-teacher/progress/*.json`.
- **Teacher cannot write the solution.** Hint ladders, not answers.
- **One lesson = one skill.** Never pack a whole course into a single skill.
- **Measure self-reliance.** Score hint count, retries, and ability to
  explain, not just correctness.

See `docs/ARCHITECTURE.md` for the full design rationale.
