---
name: writing-plans
description: Use when you have a spec or requirements for a multi-step task, before touching code
---

# Writing Plans

## Overview

Write comprehensive implementation plans assuming the engineer has zero context for our codebase and questionable taste. Document everything they need to know: which files to touch for each task, code, testing, docs they might need to check, how to test it. Give them the whole plan as bite-sized tasks. DRY. YAGNI. TDD. Frequent commits.

Assume they are a skilled developer, but know almost nothing about our toolset or problem domain. Assume they don't know good test design very well.

**Announce at start:** "I'm using the writing-plans skill to create the implementation plan."

**Context:** This should be run in a dedicated worktree (created by brainstorming skill).

**Save plans to:** `docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md`
- (User preferences for plan location override this default)

## Planning Levels

Planning happens at two levels. Each produces a different artifact.

**Initiative-level planning** produces a high-level plan: epics, user stories with acceptance criteria, file ownership per epic, and a parallelism map showing which epics can execute concurrently. This is the architect's view.

**Epic-level planning** produces a detailed implementation plan: bite-sized TDD steps with complete code for every story in the epic. This is the engineer's view.

Both levels live in the same plan document. The initiative level comes first and frames the epic-level detail that follows.

## Scope Check

If the spec covers multiple independent subsystems, it should have been broken into sub-project specs during brainstorming. If it wasn't, suggest breaking this into separate plans — one per subsystem. Each plan should produce working, testable software on its own.

## Initiative-Level Planning

### Epic Decomposition

Break the initiative into epics. Each epic is an independently deliverable unit of user value with:

- **Clear boundary** — an epic owns specific files and modules
- **Independent testability** — its tests pass without other epics being complete
- **User-facing value** — it delivers something meaningful, not just a technical layer

Bad epic: "Database Layer" (technical layer, no user value on its own)
Good epic: "Email Pattern Learning" (delivers pattern matching + confidence + feedback — testable end-to-end with mocks at the boundaries)

### User Stories

Each epic contains user stories with acceptance criteria:

```markdown
**Story:** As a [role], I want [capability] so that [benefit]

**Acceptance criteria:**
- Given [context], when [action], then [outcome]
- Given [context], when [action], then [outcome]
```

Stories within an epic are ordered by dependency. Each story maps to a commit boundary.

### File Ownership (Parallelism Contract)

Each epic must declare which files it creates or modifies. This is the parallelism contract:

```markdown
### Epic A: [Name]
**Owns:** `src/module_a.py`, `src/module_a_helpers.py`, `tests/test_module_a.py`
**Reads (does not modify):** `src/shared_types.py`, `src/config.py`
```

**The rule:** If two epics would modify the same file, they are not independent. Either:
1. Refactor the file boundary so each epic owns its piece, or
2. Serialize those epics (one depends on the other)

Shared types, config, and interfaces that multiple epics read belong in a **Foundation phase** that runs before any parallel work.

### Parallelism Map

After defining epics and their file ownership, produce a phase diagram:

```markdown
## Phases

**Phase 1 — Foundation** (serial)
Epic 0: Shared types, config, database schema, project scaffolding

**Phase 2 — Parallel Epics** (independent, zero file overlap)
Epic 1: [Name] || Epic 2: [Name]

**Phase 3 — Integration** (serial)
Epic 3: Wiring layer that connects Epic 1 + Epic 2
```

Epics in the same phase have zero file overlap — verified by the file ownership declarations. If you can't achieve zero overlap, the epics belong in different phases.

## Epic-Level Planning

Each epic gets a detailed implementation plan with the same rigor as before: bite-sized TDD steps, complete code, exact file paths, exact commands.

### File Structure

Before defining tasks, map out which files will be created or modified and what each one is responsible for. This is where decomposition decisions get locked in.

- Design units with clear boundaries and well-defined interfaces. Each file should have one clear responsibility.
- You reason best about code you can hold in context at once, and your edits are more reliable when files are focused. Prefer smaller, focused files over large ones that do too much.
- Files that change together should live together. Split by responsibility, not by technical layer.
- In existing codebases, follow established patterns. If the codebase uses large files, don't unilaterally restructure - but if a file you're modifying has grown unwieldy, including a split in the plan is reasonable.

This structure informs the task decomposition. Each task should produce self-contained changes that make sense independently.

### Bite-Sized Task Granularity

**Each step is one action (2-5 minutes):**
- "Write the failing test" - step
- "Run it to make sure it fails" - step
- "Implement the minimal code to make the test pass" - step
- "Run the tests and make sure they pass" - step
- "Commit" - step

### Plan Document Header

**Every plan MUST start with this header:**

```markdown
# [Initiative Name] Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

## Phases

**Phase 1 — Foundation** (serial)
[What must exist before parallel work begins]

**Phase 2 — Parallel Epics**
[Which epics run concurrently, with file ownership]

**Phase 3 — Integration** (serial)
[What wires the parallel work together]

---
```

### Story Structure

Within each epic, organize by user story. Each story follows the TDD task pattern:

````markdown
### Epic N: [Epic Name]

**Owns:** `exact/paths/it/creates/or/modifies`
**Reads:** `exact/paths/it/depends/on/but/does/not/modify`

#### Story N.1: [As a user, I want X so that Y]

**Acceptance criteria:**
- Given [context], when [action], then [outcome]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

- [ ] **Step 1: Write the failing test**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL with "function not defined"

- [ ] **Step 3: Write minimal implementation**

```python
def function(input):
    return expected
```

- [ ] **Step 4: Run test to verify it passes**

Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```
````

## No Placeholders

Every step must contain the actual content an engineer needs. These are **plan failures** — never write them:
- "TBD", "TODO", "implement later", "fill in details"
- "Add appropriate error handling" / "add validation" / "handle edge cases"
- "Write tests for the above" (without actual test code)
- "Similar to Story N" (repeat the code — the engineer may be reading stories out of order)
- Steps that describe what to do without showing how (code blocks required for code steps)
- References to types, functions, or methods not defined in any story

## Remember
- Exact file paths always
- Complete code in every step — if a step changes code, show the code
- Exact commands with expected output
- DRY, YAGNI, TDD, frequent commits

## Self-Review

After writing the complete plan, look at the spec with fresh eyes and check the plan against it. This is a checklist you run yourself — not a subagent dispatch.

**1. Spec coverage:** Skim each section/requirement in the spec. Can you point to a story that implements it? List any gaps.

**2. Placeholder scan:** Search your plan for red flags — any of the patterns from the "No Placeholders" section above. Fix them.

**3. Type consistency:** Do the types, method signatures, and property names you used in later stories match what you defined in earlier stories? A function called `clearLayers()` in Story 1.3 but `clearFullLayers()` in Story 2.1 is a bug.

**4. Parallelism contract:** Verify file ownership declarations. Do any two parallel epics modify the same file? If so, fix the decomposition.

If you find issues, fix them inline. No need to re-review — just fix and move on. If you find a spec requirement with no story, add the story.

## Branching Strategy

The plan must define a branching strategy that matches the phase structure:

```
main
  └── feat/<initiative-name>                      (initiative branch)
        ├── [foundation phase commits]
        ├── feat/<initiative>--epic-<name>         (epic branch, worktree)
        │     ├── story commits (direct to branch)
        │     ├── story retrospective commits (when deviating from plan)
        │     └── epic retrospective commit
        │     └── PR → feat/<initiative-name>
        ├── feat/<initiative>--epic-<name>         (parallel epic branch)
        │     └── ... same pattern ...
        │     └── PR → feat/<initiative-name>
        ├── [integration phase commits]
        └── initiative retrospective
        └── PR → main
```

**Branch naming:** Use double-dash (`--`) to separate initiative from epic names, NOT slash (`/`). Git cannot create `feat/x/y` as a branch if `feat/x` already exists as a branch ref. Epic worktree branches fork from the initiative branch tip and already contain all foundation code.

**Commit boundaries:**
- Stories commit directly to their epic branch (sequential, no conflicts)
- Each story commit includes passing tests for that story

**PR boundaries:**
- **Epic → Initiative branch:** PR with epic-level review (cross-story coherence, file ownership respected, epic retrospective included)
- **Initiative → main:** PR with full integration review (cross-epic coherence, full test suite, initiative retrospective)

## Retrospectives

Every level of the hierarchy gets a retrospective. Learnings roll up from story → epic → initiative.

**Story retrospective** — brief, appended after each story completes. **Conditional:** required when the story deviates from the plan (unexpected issues, design changes, spec gaps, API differences). Optional for stories that execute the plan verbatim with no surprises.
```markdown
#### Story N.M Retrospective
- **What worked:**
- **What didn't:**
- **Surprises / spec gaps:**
- **Plan deviations and why:**
```

**Epic retrospective** — aggregates story retros + epic-level observations. Written as the last commit on the epic branch before the PR:
```markdown
### Epic N Retrospective
**Stories completed:** N of M
**Story retro themes:** [patterns across story retros]
- **What worked at epic level:**
- **What didn't:**
- **Plan deviations:** [where the plan was wrong or incomplete]
- **Debugging time sinks:**
- **Recommendations for skills/process:**
```

**Initiative retrospective** — aggregates epic retros + cross-cutting patterns. Written before the PR to main:
```markdown
## Initiative Retrospective
**Epics completed:** N of M
**Epic retro themes:** [patterns across epic retros]
- **What worked at initiative level:**
- **What didn't:**
- **Cross-epic integration issues:**
- **Parallelism effectiveness:** [did the phase structure work? file ownership violations?]
- **Skill/process changes to encode:** [feedback worth persisting into skills or memory]
```

Save retrospectives to `docs/superpowers/retrospectives/YYYY-MM-DD-<name>.md`.

The initiative retrospective is the trigger for updating skills — if the same feedback appears across multiple epics, it should be encoded into the relevant skill rather than relying on memory.

## Issue Tracking

One GitHub issue per epic. Stories are checkboxes within the epic issue.

1. Create a GitHub issue per epic with:
   - Checkbox checklist of the epic's user stories
   - Link to the plan file
   - File ownership declaration
   - Phase assignment (foundation / parallel / integration)
2. Add all issues to the project board
3. Reference epic issues in PR descriptions and commit messages (`Closes #N`)

**Board columns:** To Do → In Progress → Done

## Execution Handoff

After saving the plan, offer execution choice:

**"Plan complete and saved to `docs/superpowers/plans/<filename>.md`. Two execution options:**

**1. Subagent-Driven (recommended)** - Dispatches agents per story, parallel epics in isolated worktrees, review between stories

**2. Inline Execution** - Execute stories in this session using executing-plans, batch execution with checkpoints

**Which approach?"**

**If Subagent-Driven chosen:**
- **REQUIRED SUB-SKILL:** Use superpowers:subagent-driven-development
- Foundation phase first, then parallel epics in worktrees, then integration

**If Inline Execution chosen:**
- **REQUIRED SUB-SKILL:** Use superpowers:executing-plans
- Batch execution with checkpoints for review
