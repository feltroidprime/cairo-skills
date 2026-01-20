---
title: Create Skill
description: Guide through creating a well-structured Claude skill with proper frontmatter and conventions
tags: [skills, meta, tooling, development]
version: 1.0.0
---

# Create Skill

You are helping the user create a new Claude skill. Follow this process carefully to ensure the skill is well-structured and follows conventions.

## Step 1: Gather Information

Ask the user these questions (you can ask multiple at once if appropriate):

1. **What does this skill do?** - Get a clear, concise description of the skill's purpose
2. **What triggers should invoke it?** - What phrases or commands should activate this skill (e.g., "review code", "write tests", "/deploy")
3. **What category does it belong to?** - Suggest categories based on purpose (e.g., "development", "testing", "documentation", "deployment", "analysis")
4. **What's a good name for it?** - Suggest a kebab-case name based on the description (e.g., "review-pr", "run-tests", "generate-docs")

## Step 2: Validate and Confirm

Before creating the skill, confirm with the user:
- The skill name (must be kebab-case, e.g., `my-skill-name`)
- The skill description (1-2 sentences)
- The triggers that will invoke it
- The category for organization

## Step 3: Create the Skill

Create the skill file at `.claude/skills/{skill-name}/SKILL.md` with this structure:

```markdown
---
title: {Title Case Name}
description: {One-line description}
tags: [{relevant, tags, here}]
version: 1.0.0
---

# {Title Case Name}

{Detailed instructions for Claude on how to perform this skill}

## When to Use

{Description of when this skill should be triggered}

## Process

{Step-by-step instructions for Claude to follow}

## Output

{What the skill should produce or accomplish}
```

### Frontmatter Requirements

- `title`: Required. Human-readable title in Title Case
- `description`: Required. One-line description (used in skill listings)
- `tags`: Required. Array of relevant tags for searchability
- `version`: Required. Semantic version starting at 1.0.0

### Content Requirements

- Clear instructions for Claude on what to do
- Specific steps or process to follow
- Expected output or outcome
- Any constraints or guidelines

## Step 4: Inform User About Next Steps

After creating the skill, tell the user:

1. The skill has been saved to `.claude/skills/{skill-name}/SKILL.md`
2. It is now available locally in this project
3. To make it available across all projects, they can use the `store-skill` skill to move it to their skills library

## Example Skill Creation

If user says "I want a skill that runs our test suite and summarizes failures", you might create:

```markdown
---
title: Run Tests
description: Execute the test suite and provide a summary of any failures
tags: [testing, automation, ci]
version: 1.0.0
---

# Run Tests

Run the project's test suite and provide a clear summary of the results.

## When to Use

Trigger this skill when the user asks to:
- Run tests
- Check if tests pass
- Verify changes don't break anything

## Process

1. Detect the project type and test framework (pytest, jest, go test, etc.)
2. Run the appropriate test command
3. Capture the output
4. If all tests pass, report success with count
5. If tests fail, summarize each failure with:
   - Test name
   - Error message
   - Relevant code context

## Output

Provide a concise summary:
- Total tests run
- Passed/failed/skipped counts
- For failures: actionable information to fix them
```
