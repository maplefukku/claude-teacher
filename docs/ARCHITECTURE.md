# claude-teacher Architecture

## Overview

`claude-teacher` is a Learning Management System built on top of Claude Code. Course authors describe a course once in a YAML DSL (`authoring/templates/course.yaml`), a pipeline of authoring skills compiles it into a runnable course pack, and learners consume that pack through a Claude Code plugin packaged under `runtime/`. At runtime the learner operates two terminals against the same repo: a read-only Teacher terminal that plans, explains, and reviews, and a normal Work terminal where the learner actually writes code. Progress, logs, and submissions live in a plain-file state directory (`.claude-teacher/`) so the system degrades gracefully, is inspectable without any database, and can be diffed and committed like any other artifact. The architectural line to keep in your head: authoring produces data, runtime consumes data, and state is always a file on disk.

## The core separation

Claude Code exposes five extension points, and `claude-teacher` uses each for exactly one job:

- **Skill**: a procedural, author-invoked capability. In `claude-teacher`, skills are the authoring pipeline steps (`/design-course`, `/compile-course-pack`, etc.) and the runtime verbs the learner triggers (`course-start`, `checkpoint`).
- **Subagent**: a scoped persona with its own tool allowlist. `claude-teacher` ships one, `runtime/agents/teacher-coach.md`, used as the teacher voice in the Teacher terminal and restricted to read-only tools.
- **Hook**: deterministic shell code the harness runs around tool calls. Used here to enforce read-only boundaries, log tool use, and auto-write progress after submission edits.
- **Plugin**: the unit of distribution. `runtime/` is packaged as a Claude Code plugin so learners install the entire runtime (skills + agent + courses) with one command.
- **Managed settings**: org-level `settings.json` pushed by an admin. Used in the final distribution phase to pin the plugin version and lock hook configuration on fleet machines.

## Directory layout

```
claude-teacher/
├── authoring/                 # Author-side DSL, templates, and the compile pipeline
│   ├── .claude/skills/        # /design-course, /design-module, /design-lesson,
│   │                          # /compile-course-pack, /review-course-quality
│   └── templates/course.yaml  # Canonical course DSL template
├── runtime/                   # The shipped Claude Code plugin (learner-side)
│   ├── .claude-plugin/plugin.json
│   ├── skills/                # course-start, lesson-start, checkpoint,
│   │                          # give-hint, review-submission, reflect
│   ├── agents/teacher-coach.md
│   └── courses/<course-id>/   # Compiled course packs: manifest.json + lessons/
├── learner-workspace/         # Example learner repo; hooks live in .claude/settings.json
├── docs/                      # Architecture and design notes (this file)
└── README.md                  # Project entry point
```

At runtime the learner also has a sibling directory `.claude-teacher/` (outside the plugin, typically at the workspace root) holding `progress/`, `logs/`, `submissions/`, and `state/schema.json`. That directory is considered learner data and is gitignored in templates, though nothing prevents a learner from committing it to their own fork if they want a permanent record of their journey.

Note the asymmetry between `authoring/.claude/skills/` and `runtime/skills/`: authoring skills are project-scoped and only activate when the author is inside the repo, while runtime skills are plugin-scoped and activate wherever the plugin is installed. This is deliberate — authors should not accidentally see runtime skills while building a course, and learners should never see the authoring pipeline at all.

## Authoring pipeline

Authors never hand-edit `SKILL.md` files. They edit `course.yaml` and run a linear chain of skills. Each skill reads the previous artifact and produces the next.

| Skill                     | Input                              | Output artifact                                  |
|---------------------------|------------------------------------|--------------------------------------------------|
| `/design-course`          | Raw idea + `templates/course.yaml` | `course.yaml` with course-level goals filled in  |
| `/design-module`          | `course.yaml` goals                | `course.yaml` with a modules[] section           |
| `/design-lesson`          | A single module entry              | Per-lesson spec blocks inside `course.yaml`      |
| `/compile-course-pack`    | Finalized `course.yaml`            | `runtime/courses/<course-id>/` manifest + lessons|
| `/review-course-quality`  | Compiled course pack               | Quality report + diff suggestions                |

Data flow:

```
  idea
    |
    v
 [ /design-course ] ---> course.yaml (goals)
    |
    v
 [ /design-module ] ---> course.yaml (modules[])
    |
    v
 [ /design-lesson ] ---> course.yaml (lessons[])
    |
    v
 [ /compile-course-pack ] ---> runtime/courses/<course-id>/
    |                              manifest.json
    |                              lessons/<id>.md
    v
 [ /review-course-quality ] ---> report + suggested edits
                                 (loops back into course.yaml)
```

The DSL is the source of truth. Compiled packs under `runtime/courses/` are regeneratable and should never be edited by hand. If `/review-course-quality` finds a problem, the fix lands in `course.yaml` and the pack is recompiled; this keeps the authoring and runtime sides from drifting.

Each stage is a distinct skill rather than one monolithic `/build-course` for two reasons. First, authors need to iterate at different granularities: polishing a single lesson should not re-run module design. Second, each stage has a different "done" criterion and therefore a different reviewer prompt; collapsing them into one skill would bloat `SKILL.md` and make failures hard to localize.

## Runtime surface

The plugin in `runtime/` exposes six learner-facing skills plus one subagent:

- `course-start`: load a course pack from `runtime/courses/<id>/`, initialize `.claude-teacher/progress/<id>/`, and announce the first lesson.
- `lesson-start`: open the current lesson, render objectives, and seed the Claude Code task list for in-session UI.
- `checkpoint`: verify that the learner has met a lesson's exit criteria; records a checkpoint entry in progress JSON.
- `give-hint`: produce the next smallest hint for the current step without revealing the solution.
- `review-submission`: read files in `.claude-teacher/submissions/`, grade against the lesson rubric, and append a review record.
- `reflect`: end-of-lesson retrospective that writes a reflection entry into progress JSON and advances to the next lesson.

The subagent `runtime/agents/teacher-coach.md` is the persona used whenever the Teacher terminal speaks. Its frontmatter restricts it to read-only tools (`Read`, `Grep`, `Glob`, and `Bash` limited to read commands such as `ls`, `cat`, `git status`, and test runners) so it physically cannot edit learner code, even if a skill invocation goes wrong. Its system prompt also forbids it from quoting more than a few lines of a candidate solution back to the learner — the goal is to coach, not to leak.

## Two-terminal model

The learner runs two Claude Code sessions against the same repo, simultaneously:

1. **Teacher Terminal** — launched with the `teacher-coach` subagent and typically in a plan-mode-style configuration. It can read the repo, read `.claude-teacher/`, run test commands, and speak. It cannot `Write`, `Edit`, or run destructive bash. This is where `/lesson-start`, `/give-hint`, `/review-submission`, and `/reflect` are invoked.
2. **Work Terminal** — a normal Claude Code session. The learner types prompts here and edits code here. Its hooks log activity into `.claude-teacher/logs/` and mirror submissions into `.claude-teacher/submissions/`.

Why not one terminal? Pedagogy requires a persistent teacher voice whose state is not poisoned by every experiment the learner runs. If the teacher and the worker share a context window, the teacher starts "seeing" the learner's half-finished code as ground truth and begins to solve the problem instead of teaching it — the classic "autocomplete tutor" failure mode. Separating the terminals also lets us apply different tool permissions cleanly: the Teacher is read-only by construction, so we get safety from the harness rather than from prompt discipline. Finally, the two sessions can be compacted and cleared independently: the learner can `/clear` their Work terminal between experiments without losing the teacher's threaded understanding of where they are in the course.

## State model

All durable state lives in `.claude-teacher/` as flat files. The canonical shape of a per-lesson progress file is:

```json
{
  "course_id": "intro-to-claude-code",
  "lesson_id": "02-first-skill",
  "status": "in_progress",
  "started_at": "2026-04-14T09:12:03Z",
  "updated_at": "2026-04-14T09:47:51Z",
  "completed_at": null,
  "attempts": 2,
  "checkpoints": [
    { "id": "c1", "passed": true,  "at": "2026-04-14T09:22:10Z" },
    { "id": "c2", "passed": false, "at": "2026-04-14T09:40:02Z" }
  ],
  "hints_used": 1,
  "submissions": [
    { "path": ".claude-teacher/submissions/02-first-skill/skill.md",
      "hash": "sha256:...", "at": "2026-04-14T09:39:50Z" }
  ],
  "reflection": null,
  "schema_version": 1
}
```

Fields:

- `course_id`, `lesson_id`: stable identifiers matching `runtime/courses/<course-id>/manifest.json`.
- `status`: one of `not_started`, `in_progress`, `completed`, `abandoned`.
- `started_at` / `updated_at` / `completed_at`: ISO-8601 UTC timestamps.
- `attempts`: incremented each time the learner re-enters the lesson.
- `checkpoints[]`: append-only log of exit-criteria evaluations from `/checkpoint`.
- `hints_used`: counter bumped by `/give-hint`; surfaces in reviews.
- `submissions[]`: pointer records for files copied into `.claude-teacher/submissions/` by the Work terminal's hook. Hashes let the teacher detect silent rewrites.
- `reflection`: populated by `/reflect` at lesson end.
- `schema_version`: validated against `.claude-teacher/state/schema.json` on every write.

Claude Code's in-session task list is used for UI only — crossing items off, showing the learner where they are in a lesson. It is ephemeral, per-session, and invisible to the Teacher terminal. Treating the task list as the progress DB would mean losing all state on `/clear`, making cross-terminal coordination impossible, and having no audit trail for `/review-submission`. Progress JSON is the source of truth; task lists are a view derived from it.

Writes to progress JSON are append-mostly: `checkpoints[]` and `submissions[]` never rewrite history, and the only mutable scalars are `status`, `updated_at`, `completed_at`, `attempts`, and `hints_used`. This makes the files safe to read concurrently from the Teacher terminal while the Work terminal is writing to them, and makes `git diff` on `.claude-teacher/` a legible account of what the learner did.

## Hooks

Hooks are configured in `learner-workspace/.claude/settings.json` and enforced by the harness, not the model:

- **`PreToolUse` log hook**: fires before every tool call in the Work terminal, appending a JSONL record to `.claude-teacher/logs/<date>.jsonl`. Runs regardless of tool outcome; gives the teacher a replayable trace.
- **`PreToolUse` guard hook**: fires before `Write` and `Edit`. Blocks any path under `runtime/`, `authoring/`, or `.claude-teacher/progress/`. These are teacher-owned; the learner must not mutate them from the Work terminal. Blocked calls surface an error to the model so it self-corrects.
- **`PostToolUse` progress hook**: fires after successful `Write`/`Edit` whose path is under `.claude-teacher/submissions/`. It computes a content hash, appends a record to the current lesson's progress JSON, and bumps `updated_at`. This is how progress advances without the model having to remember to write it.

The Teacher terminal uses the same `settings.json` but additionally runs under the `teacher-coach` subagent, which strips `Write`/`Edit` entirely, so the guard hook is a belt-and-suspenders defense.

Hook scripts themselves live under `learner-workspace/.claude/hooks/` as small shell or Python files invoked by absolute path. Keeping them out of `runtime/` means they can be audited and signed independently of the plugin, and a managed-settings rollout can replace them without touching the plugin version. Hooks intentionally do not call back into Claude — they are deterministic and side-effect-scoped, so a hook failure never blocks the learner from making progress; the worst case is a missing log line.

## Distribution

Distribution rolls out in three phases, each broader than the last:

1. **Project skills (today)**: `runtime/` is loaded as project-local skills by pointing Claude Code at the repo. This is the inner loop — authors and early learners clone the repo and work directly against `runtime/`.
2. **Plugin**: `runtime/.claude-plugin/plugin.json` defines the plugin manifest. The same directory is published so a learner can `claude plugin install claude-teacher` and get skills, the teacher-coach agent, and all compiled courses atomically. `runtime/` in the repo remains the working tree; published versions are tags.
3. **Managed settings**: for classroom or org rollouts, an admin pushes a managed `settings.json` that pins the plugin version, forces the hook configuration, and locks the teacher-coach tool allowlist. Learners cannot disable the guard hooks even by editing their local settings, and a central update flips everyone's course library at once.

The three phases are additive. Phase 1 is the developer inner loop, phase 2 is the individual learner install, and phase 3 is fleet deployment. At no point does the content of `runtime/` change between phases — only how it is delivered. That invariant is why the repo keeps `runtime/` as a first-class working tree instead of hiding it in a build output.

## Anti-patterns

Things `claude-teacher` deliberately does NOT do:

- **One skill per course.** Courses are data, not skills. A new course is a new entry in `runtime/courses/`, never a new directory in `runtime/skills/`.
- **Carpet-bomb `CLAUDE.md`.** Teaching content belongs in lesson files under `runtime/courses/<id>/lessons/`, not smeared across project memory where it leaks into every unrelated session.
- **Let the Teacher terminal edit code.** The teacher-coach agent is read-only by design. If you find yourself wanting to give it `Write`, you are solving the learner's problem for them.
- **Trust the Claude Code task list as the progress DB.** Task lists are per-session UI. Progress must round-trip through `.claude-teacher/progress/`.
- **Pack everything into the `SKILL.md` body.** `SKILL.md` is a thin entry point; heavy content lives in sibling files referenced by path. This keeps skill activation cheap and makes diffs reviewable.
- **Reach across the terminal boundary.** The Teacher terminal never writes to the Work terminal's session and vice versa. All cross-terminal communication goes through files in `.claude-teacher/`.
- **Hand-edit compiled course packs.** If a lesson is wrong, fix `course.yaml` and re-run `/compile-course-pack`. Editing `runtime/courses/<id>/lessons/` directly will be silently clobbered on the next compile.

## MVP scope

In the initial build:

- The authoring DSL in `authoring/templates/course.yaml` and all five authoring skills under `authoring/.claude/skills/`.
- The runtime plugin under `runtime/` with all six learner skills, the `teacher-coach` subagent, and at least one compiled course under `runtime/courses/`.
- The two-terminal workflow documented and usable against `learner-workspace/`.
- Hooks in `learner-workspace/.claude/settings.json` for logging, edit-guarding, and progress writes.
- `.claude-teacher/` state with JSON schema validation at `.claude-teacher/state/schema.json`.

Explicitly deferred:

- Plugin publication and managed-settings rollout (phases 2 and 3 of Distribution).
- A web dashboard over `.claude-teacher/progress/`.
- Multi-learner or classroom aggregation.
- Localization of course content and teacher persona.
- Automatic regrading when a course pack is recompiled against existing progress.
- Rich media in lessons (images, video); MVP lessons are Markdown-only.
- A learner-facing CLI separate from Claude Code. For MVP, the only entry points are the six runtime skills.

If you are reading this file after a clone and want to start somewhere concrete: open `authoring/templates/course.yaml` to see the input, open `runtime/courses/` for an example of the output, and read `runtime/agents/teacher-coach.md` to understand the voice the learner will hear. Everything else in this document is there to defend those three files.

When in doubt, the architectural test for any proposed change is: "Would this make authoring, runtime, and state bleed into each other?" If yes, it does not belong in `claude-teacher`.
