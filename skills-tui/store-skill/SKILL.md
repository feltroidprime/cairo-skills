---
title: Store Skill
description: Move a local skill from .claude/skills/ to the central skills repository
tags: [skills, meta, tooling, organization]
version: 1.0.0
---

# Store Skill

You are helping the user move a locally-created skill to their central skills repository so it can be installed in other projects.

## Step 1: Find Local Skills

Look for skills in `.claude/skills/` that are regular directories (not symlinks). These are locally-created skills that could be stored in the repository.

```bash
# List directories in .claude/skills that are NOT symlinks
find .claude/skills -maxdepth 1 -type d ! -name "skills" | while read dir; do
  if [ ! -L "$dir" ]; then
    basename "$dir"
  fi
done
```

If no local skills are found, inform the user:
- They don't have any local skills to store
- They can create one using the `create-skill` skill
- Or manually create a skill in `.claude/skills/{name}/SKILL.md`

## Step 2: Get the Skills Repository Path

Read the skills-tui configuration to find the repository path:

```bash
cat ~/.config/skills-tui/config.json
```

Look for the `skills_path` key. This is where skills should be stored.

If no config exists, ask the user to either:
1. Run `skills` to configure the path
2. Provide the path directly

## Step 3: Select Skill and Category

If multiple local skills exist, ask the user which one to store.

Then ask for the category:
- Suggest a category based on the skill's tags (from its frontmatter)
- Common categories: `development`, `testing`, `documentation`, `deployment`, `analysis`, `git`, `utilities`
- Category should be kebab-case

## Step 4: Move the Skill

Move the skill to the repository:

```bash
# Create category directory if needed
mkdir -p {skills_path}/{category}

# Move the skill
mv .claude/skills/{skill-name} {skills_path}/{category}/{skill-name}
```

## Step 5: Offer to Keep Installed

Ask the user if they want to keep the skill installed in the current project:
- **Yes**: Create a symlink from `.claude/skills/{skill-name}` to the new location
- **No**: The skill is now only in the repository and must be re-installed via skills-tui

```bash
# If keeping installed, create symlink
ln -s {skills_path}/{category}/{skill-name} .claude/skills/{skill-name}
```

## Step 6: Confirm Success

Tell the user:
1. The skill has been moved to `{skills_path}/{category}/{skill-name}/`
2. It will now appear in skills-tui under the `{category}` category
3. They can install it in other projects using the `skills` command

## Example Workflow

User: "Store my run-tests skill"

1. Check `.claude/skills/run-tests` exists and is not a symlink
2. Read `~/.config/skills-tui/config.json` to get `skills_path` (e.g., `~/skills`)
3. Ask: "What category should this go in? Based on the tags [testing, automation], I suggest 'testing'."
4. User confirms: "testing"
5. Run: `mkdir -p ~/skills/testing && mv .claude/skills/run-tests ~/skills/testing/`
6. Ask: "Would you like to keep it installed in this project?"
7. If yes: `ln -s ~/skills/testing/run-tests .claude/skills/run-tests`
8. Confirm: "Done! Your skill is now stored at ~/skills/testing/run-tests/ and will appear in skills-tui."

## Error Handling

- If the skill already exists in the repository at that location, ask if they want to overwrite
- If the symlink creation fails, warn but don't fail - the skill is still stored
- If the skills_path doesn't exist, ask if they want to create it
