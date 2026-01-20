---
title: Ralph TUI Create Beads
description: Convert PRDs to beads for ralph-tui execution. Creates an epic with child beads for each user story.
tags:
  - ralph
  - beads
  - task-management
  - automation
version: 1.0.0
---

# Ralph TUI - Create Beads

Converts PRDs to beads (epic + child tasks) for ralph-tui autonomous execution.

> **Note:** This skill is bundled with ralph-tui's Beads tracker plugin. Future tracker plugins (Linear, GitHub Issues, etc.) will bundle their own task creation skills.

---

## The Job

Take a PRD (markdown file or text) and create beads in `.beads/beads.jsonl`:
1. **Extract Quality Gates** from the PRD's "Quality Gates" section
2. Create an **epic** bead for the feature
3. Create **child beads** for each user story (with quality gates appended)
4. Set up **dependencies** between beads (schema → backend → UI)
5. Output ready for `ralph-tui run --tracker beads`

---

## Step 1: Extract Quality Gates

Look for the "Quality Gates" section in the PRD:

```markdown
## Quality Gates

These commands must pass for every user story:
- `pnpm typecheck` - Type checking
- `pnpm lint` - Linting

For UI stories, also include:
- Verify in browser using dev-browser skill
```

Extract:
- **Universal gates:** Commands that apply to ALL stories (e.g., `pnpm typecheck`)
- **UI gates:** Commands that apply only to UI stories (e.g., browser verification)

**If no Quality Gates section exists:** Ask the user what commands should pass, or use a sensible default like `npm run typecheck`.

---

## Output Format

Beads use `bd create` command:

```bash
# Create epic (link back to source PRD)
bd create --type=epic \
  --title="[Feature Name]" \
  --description="[Feature description from PRD]" \
  --external-ref="prd:./tasks/feature-name-prd.md" \
  --labels="ralph,feature"

# Create child bead (with quality gates in acceptance criteria)
bd create \
  --parent=EPIC_ID \
  --title="[Story Title]" \
  --description="[Story description with acceptance criteria INCLUDING quality gates]" \
  --priority=[1-4] \
  --labels="ralph,task"
```

---

## Story Size: The #1 Rule

**Each story must be completable in ONE ralph-tui iteration (~one agent context window).**

ralph-tui spawns a fresh agent instance per iteration with no memory of previous work. If a story is too big, the agent runs out of context before finishing.

### Right-sized stories:
- Add a database column + migration
- Add a UI component to an existing page
- Update a server action with new logic
- Add a filter dropdown to a list

### Too big (split these):
- "Build the entire dashboard" → Split into: schema, queries, UI components, filters
- "Add authentication" → Split into: schema, middleware, login UI, session handling
- "Refactor the API" → Split into one story per endpoint or pattern

**Rule of thumb:** If you can't describe the change in 2-3 sentences, it's too big.

---

## Story Ordering: Dependencies First

Stories execute in dependency order. Earlier stories must not depend on later ones.

**Correct order:**
1. Schema/database changes (migrations)
2. Server actions / backend logic
3. UI components that use the backend
4. Dashboard/summary views that aggregate data

**Wrong order:**
1. ❌ UI component (depends on schema that doesn't exist yet)
2. ❌ Schema change

---

## Dependencies with `bd dep add`

Use the `bd dep add` command to specify which beads must complete first:

```bash
# Create the beads first
bd create --parent=epic-123 --title="US-001: Add schema" ...
bd create --parent=epic-123 --title="US-002: Create API" ...
bd create --parent=epic-123 --title="US-003: Build UI" ...

# Then add dependencies (issue depends-on blocker)
bd dep add ralph-tui-002 ralph-tui-001  # US-002 depends on US-001
bd dep add ralph-tui-003 ralph-tui-002  # US-003 depends on US-002
```

**Syntax:** `bd dep add <issue> <depends-on>` — the issue depends on (is blocked by) depends-on.

ralph-tui will:
- Show blocked beads as "blocked" until dependencies complete
- Never select a bead for execution while its dependencies are open
- Include dependency context in the prompt when working on a bead

---

## Conversion Rules

1. **Extract Quality Gates** from PRD first
2. **Each user story → one bead**
3. **First story**: No dependencies (creates foundation)
4. **Subsequent stories**: Depend on their predecessors (UI depends on backend, etc.)
5. **Priority**: Based on dependency order, then document order (0=critical, 2=medium, 4=backlog)
6. **Labels**: Epic gets `ralph,feature`; Tasks get `ralph,task`
7. **All stories**: `status: "open"`
8. **Acceptance criteria**: Story criteria + quality gates appended
9. **UI stories**: Also append UI-specific gates (browser verification)

---

## Output Location

Beads are written to: `.beads/beads.jsonl`

After creation, run ralph-tui:
```bash
# Work on a specific epic
ralph-tui run --tracker beads --epic ralph-tui-abc

# Or let it pick the best task automatically
ralph-tui run --tracker beads
```

---

## Checklist Before Creating Beads

- [ ] Extracted Quality Gates from PRD (or asked user if missing)
- [ ] Each story is completable in one iteration (small enough)
- [ ] Stories are ordered by dependency (schema → backend → UI)
- [ ] Quality gates appended to every bead's acceptance criteria
- [ ] UI stories have browser verification (if specified in Quality Gates)
- [ ] Acceptance criteria are verifiable (not vague)
- [ ] No story depends on a later story (only earlier stories)
- [ ] Dependencies added with `bd dep add` after creating beads
