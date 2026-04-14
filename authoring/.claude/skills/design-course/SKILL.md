---
name: design-course
description: Interview a course author and fill in the course-level section of course.yaml for a new claude-teacher course. Use this as the first step of authoring a new course, before any modules or lessons are designed. Outputs a course.yaml skeleton that /design-module and /design-lesson will extend.
argument-hint: [course-id]
allowed-tools: Read, Write, Edit, Glob
---

# design-course

You are helping a course author define the top of a new course. You are NOT
designing modules or lessons here — you are defining what the course IS and
what it is NOT. That scoping is the most valuable thing an author can do,
and it is the thing people skip fastest. Do not let them skip it.

## Arguments

- `$1` — course id (kebab-case, e.g. `claude-code-hooks-intro`). If missing,
  ask.

## Steps

1. **Create the output directory.** If
   `authoring/outputs/$1/course.yaml` does not exist, copy
   `authoring/templates/course.yaml` there as a starting skeleton. If it
   already exists, load it and continue filling in missing fields rather than
   overwriting.

2. **Run the interview.** Ask the following questions one at a time, waiting
   for each answer before moving on. Do NOT batch them. Batching turns an
   interview into a survey and you get survey-quality answers.

   1. "Who is this course for? Be specific. 'Engineers' is not specific.
      'Backend engineers new to Claude Code who have never written a Skill'
      is specific."
   2. "What will they be able to DO after this course that they cannot do
      now? Give me up to 5 verbs."
   3. "What will they still NOT be able to do? Non-goals are as important as
      goals — they keep the scope honest."
   4. "What must they already know coming in? List prerequisites as concrete
      checkable items, not 'general familiarity with X'."
   5. "How will you know the course worked? Describe the completion
      criteria. 'They finished all lessons' is not enough — what artifact
      proves it?"
   6. "How long do you want this to take, total? Be realistic. Learners
      always take 1.5x–2x what authors estimate."
   7. "Grading strategy: per-lesson rubric with /checkpoint, or something
      else?"

3. **Write the fields back into `course.yaml`.** Fill in `course.id`,
   `course.title`, `course.audience`, `course.goal`, `course.non_goals`,
   `course.prerequisites`, `course.estimated_duration_min`,
   `course.completion_criteria`, `course.assessment_strategy`. Do NOT touch
   the `modules:` or `lessons:` lists — those belong to `/design-module` and
   `/design-lesson`.

4. **Stress-test the scope.** After the fields are filled in, read them back
   to the author and ask three challenge questions:
   - "If you could cut one goal, which one and why?"
   - "Which non-goal is likeliest to be demanded by a learner mid-course,
     and how will you say no?"
   - "Can the completion criteria be checked without a human present?"
   Record the answers as a `## Scope review` comment block at the bottom of
   `course.yaml`. This is the paper trail.

5. **Hand off.** Tell the author the exact next command:
   `/design-module $1`.

## Output contract

- `authoring/outputs/$1/course.yaml` exists and has all course-level fields
  populated.
- `modules:` and `lessons:` lists are present but empty (or unchanged).
- A scope review comment is appended.

## Refusals

- Do NOT invent audience, goals, or non-goals. If the author gives a vague
  answer, reflect it back with "can you be more specific" until it is
  concrete. A vague course.yaml is a broken course.
- Do NOT proceed to module design in this skill.
- Do NOT estimate duration for the author.
