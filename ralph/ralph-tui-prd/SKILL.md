---
title: Ralph TUI PRD Generator
description: Generate a Product Requirements Document (PRD) for ralph-tui task orchestration
tags:
  - ralph
  - prd
  - planning
  - requirements
version: 1.0.0
---

# Ralph TUI PRD Generator

Create detailed Product Requirements Documents optimized for AI agent execution via ralph-tui.

---

## The Job

1. Receive a feature description from the user
2. Ask 3-5 essential clarifying questions (with lettered options) - one set at a time
3. **Always ask about quality gates** (what commands must pass)
4. After each answer, ask follow-up questions if needed (adaptive exploration)
5. Generate a structured PRD when you have enough context
6. Output the PRD wrapped in `[PRD]...[/PRD]` markers for TUI parsing

**Important:** Do NOT start implementing. Just create the PRD.

---

## Step 1: Clarifying Questions (Iterative)

Ask questions one set at a time. Each answer should inform your next questions. Focus on:

- **Problem/Goal:** What problem does this solve?
- **Core Functionality:** What are the key actions?
- **Scope/Boundaries:** What should it NOT do?
- **Success Criteria:** How do we know it's done?
- **Integration:** How does it fit with existing features?
- **Quality Gates:** What commands must pass for each story? (REQUIRED)

### Format Questions Like This:

```
1. What is the primary goal of this feature?
   A. Improve user onboarding experience
   B. Increase user retention
   C. Reduce support burden
   D. Other: [please specify]

2. Who is the target user?
   A. New users only
   B. Existing users only
   C. All users
   D. Admin users only
```

This lets users respond with "1A, 2C" for quick iteration.

### Quality Gates Question (REQUIRED)

Always ask about quality gates - these are project-specific:

```
What quality commands must pass for each user story?
   A. pnpm typecheck && pnpm lint
   B. npm run typecheck && npm run lint
   C. bun run typecheck && bun run lint
   D. Other: [specify your commands]

For UI stories, should we include browser verification?
   A. Yes, use dev-browser skill to verify visually
   B. No, automated tests are sufficient
```

### Adaptive Questioning

After each response, decide whether to:
- Ask follow-up questions (if answers reveal complexity)
- Ask about a new aspect (if current area is clear)
- Generate the PRD (if you have enough context)

Typically 2-4 rounds of questions are needed.

---

## Step 2: PRD Structure

Generate the PRD with these sections:

### 1. Introduction/Overview
Brief description of the feature and the problem it solves.

### 2. Goals
Specific, measurable objectives (bullet list).

### 3. Quality Gates
**CRITICAL:** List the commands that must pass for every user story.

```markdown
## Quality Gates

These commands must pass for every user story:
- `pnpm typecheck` - Type checking
- `pnpm lint` - Linting

For UI stories, also include:
- Verify in browser using dev-browser skill
```

This section is extracted by conversion tools (ralph-tui-create-json, ralph-tui-create-beads) and appended to each story's acceptance criteria.

### 4. User Stories
Each story needs:
- **Title:** Short descriptive name
- **Description:** "As a [user], I want [feature] so that [benefit]"
- **Acceptance Criteria:** Verifiable checklist of what "done" means

Each story should be small enough to implement in one focused AI agent session.

**Format:**
```markdown
### US-001: [Title]
**Description:** As a [user], I want [feature] so that [benefit].

**Acceptance Criteria:**
- [ ] Specific verifiable criterion
- [ ] Another criterion
```

**Note:** Do NOT include quality gate commands in individual story criteria - they are defined once in the Quality Gates section and applied automatically during conversion.

**Important:**
- Acceptance criteria must be verifiable, not vague
- "Works correctly" is bad
- "Button shows confirmation dialog before deleting" is good
- Each story should be independently completable

### 5. Functional Requirements
Numbered list of specific functionalities:
- "FR-1: The system must allow users to..."
- "FR-2: When a user clicks X, the system must..."

Be explicit and unambiguous.

### 6. Non-Goals (Out of Scope)
What this feature will NOT include. Critical for managing scope.

### 7. Technical Considerations (Optional)
- Known constraints or dependencies
- Integration points with existing systems
- Performance requirements

### 8. Success Metrics
How will success be measured?

### 9. Open Questions
Remaining questions or areas needing clarification.

---

## Writing for AI Agents

The PRD will be executed by AI coding agents via ralph-tui. Therefore:

- Be explicit and unambiguous
- User stories should be small (completable in one session)
- Acceptance criteria must be machine-verifiable where possible
- Include specific file paths if you know them
- Reference existing code patterns in the project

---

## Output Format

**CRITICAL:** Wrap the final PRD in markers for TUI parsing:

```
[PRD]
# PRD: Feature Name

## Overview
...

## Quality Gates
...

## User Stories
...
[/PRD]
```

**File naming:** The TUI will save to `./tasks/prd-[feature-name].md`

---

## Checklist

Before outputting the PRD:

- [ ] Asked clarifying questions with lettered options
- [ ] Asked about quality gates (REQUIRED)
- [ ] Asked follow-up questions when needed
- [ ] Quality Gates section included with project-specific commands
- [ ] User stories are small and independently completable
- [ ] User stories do NOT include quality gate commands (they're in the Quality Gates section)
- [ ] Functional requirements are numbered and unambiguous
- [ ] Non-goals section defines clear boundaries
- [ ] PRD is wrapped in `[PRD]...[/PRD]` markers
