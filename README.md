# claude-teacher

Interactive hands-on LMS that runs inside Claude Code.

> 日本語版の README は [README.ja.md](./README.ja.md) を参照してください。

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

## Setup

### Prerequisites

- [Claude Code](https://docs.claude.com/en/docs/claude-code) installed (CLI,
  desktop, or IDE extension) and authenticated.
- `git` on your `PATH`.
- A shell that lets you keep two Claude Code sessions open side by side
  (two terminal panes, two tmux windows, two IDE panels — anything works).

### 1. Clone the repository

```bash
git clone https://github.com/maplefukku/claude-teacher.git
cd claude-teacher
```

### 2. Prepare a learner workspace

You need a directory where the Work Terminal will live. You have two options.

**Option A — use the bundled example (fastest way to try it out).**
The `learner-workspace/` directory already ships with a working
`.claude/settings.json` that wires up the logging, edit-guard, and
progress-writing hooks described in `docs/ARCHITECTURE.md`.

```bash
cd learner-workspace
```

**Option B — bootstrap a fresh project.**

```bash
mkdir my-learning-project
cd my-learning-project
mkdir -p .claude
cp ../claude-teacher/learner-workspace/.claude/settings.json .claude/settings.json
```

The `SessionStart` hook inside that `settings.json` will create the
`.claude-teacher/{progress,logs,submissions,state}` directories the first
time you launch Claude Code in the workspace, so you do not need to
`mkdir` them yourself.

### 3. Load the `runtime/` plugin

The runtime is shipped as a Claude Code plugin under
`runtime/.claude-plugin/`. Until it is published to a plugin registry you
load it as a project-local plugin (distribution phase 1 in
`docs/ARCHITECTURE.md`):

- **Quickest:** launch Claude Code from the root of this repo so that
  `runtime/skills/` and `runtime/agents/teacher-coach.md` are discovered as
  project-scoped components.
- **From a separate learner workspace:** point Claude Code's plugin
  configuration at the absolute path of this repo's `runtime/` directory,
  or symlink `runtime/.claude-plugin` into your workspace's `.claude/`
  directory.

Once the plugin is loaded you should see the six learner skills
(`course-start`, `lesson-start`, `checkpoint`, `give-hint`,
`review-submission`, `reflect`) and the `teacher-coach` subagent available
inside Claude Code.

### 4. Open the two terminals

Open **two** Claude Code sessions against the same learner workspace:

| Terminal         | Launch as                                  | What you do there                         |
| ---------------- | ------------------------------------------ | ----------------------------------------- |
| Teacher Terminal | plan mode + `teacher-coach` subagent       | `/course-start`, `/lesson-start`, review  |
| Work Terminal    | normal Claude Code session                 | Write code, run commands, break things    |

The Teacher Terminal is read-only by construction — the `teacher-coach`
subagent strips `Write`/`Edit` from its tool allowlist, and the
`PreToolUse` guard hook blocks edits to `runtime/`, `authoring/`, and
`.claude-teacher/progress/` even if something slips through.

### 5. Start a course

In the **Teacher Terminal**, kick off the bundled sample course:

```
/course-start claude-code-skills-basic
```

Then, in the **Work Terminal**, follow the instructions the teacher prints.
The verbs you will use most often:

- `/lesson-start` — open the next lesson and seed the in-session task list.
- `/give-hint` — ask for the next smallest hint when you are stuck.
- `/checkpoint` — verify a lesson's exit criteria once you think you are done.
- `/review-submission` — have the teacher grade the files in
  `.claude-teacher/submissions/` against the lesson rubric.
- `/reflect` — close out the lesson and advance to the next one.

All progress is written to `.claude-teacher/progress/<course-id>/` as plain
JSON, so you can `git diff` your learning journey or start over by deleting
the directory.

## Authoring a new course

If instead of *taking* a course you want to *build* one, work inside the
`authoring/` project rather than `learner-workspace/`. Launch Claude Code
from `authoring/` so its project-scoped skills are picked up, then edit
`authoring/templates/course.yaml` as your source of truth and run the
pipeline:

1. `/design-course` — hearing interview, fills in `course.yaml`.
2. `/design-module` — module-level breakdown.
3. `/design-lesson` — per-lesson deep dive.
4. `/compile-course-pack` — emits a runnable course pack under
   `runtime/courses/<course-id>/`.
5. `/review-course-quality` — internal QA pass before shipping.

Compiled packs in `runtime/courses/` are then consumed by the learner
runtime described in the Setup section above.

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
