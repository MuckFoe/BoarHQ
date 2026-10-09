---
name: sdd-tasks
description: Decompose an approved plan.md into a numbered, context-sized task list (tasks.md) with acceptance criteria and dependencies. Use when asked to break a plan into tasks, after plan.md has been approved.
---

# sdd-tasks

Input: `specs/<slug>/plan.md` + `specs/<slug>/spec.md`.
Output: `specs/<slug>/tasks.md`.

## Context discipline

Read only this feature's plan.md and spec.md. Nothing else in `specs/`.

## Steps

1. Decompose the plan into numbered tasks. Each task:
   - One cohesive change — touches roughly ≤3 files, one concern. If it doesn't fit, split it further, don't hand an oversized task to `/sdd-implement`.
   - Explicit **acceptance criteria**, lifted from spec.md's Given/When/Then for the requirement it implements.
   - Explicit **inputs/dependencies** (which prior task must land first) and files touched.
   - Flag `[P]` if it has no dependency on any other unfinished task in this list (safe to run in the same wave).
2. Order tasks so dependent work follows its prerequisite; group `[P]`-flagged tasks into waves.
3. Every task must trace to a plan.md component and a spec.md requirement. Don't add tasks for anything not in the plan.
4. Stop. Print the file path and: **"Review, reorder, or split tasks.md, then run /sdd-implement one task (or one [P] wave) at a time."**

## Do / Don't

**Do:** size for a single agent context window; carry acceptance criteria forward verbatim where possible so `/sdd-implement` can write the test straight from it.

**Don't:** implement anything here. Don't create tasks with vague criteria like "make it work."
