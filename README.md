# Cairo Skills for Claude Code

A Claude Code plugin with skills for profiling and developing Cairo programs.

## Available Skills

| Skill | Description |
|-------|-------------|
| [benchmarking-cairo](skills/benchmarking-cairo/) | Profile Cairo functions with cairo-profiler and pprof |
| [using-bounded-int](skills/using-bounded-int/) | Optimize Cairo arithmetic with bounded integers |

## Installation

### As a Claude Code Plugin

Clone to your preferred location and install as a plugin:

```bash
git clone https://github.com/feltroidprime/cairo-skills.git ~/cairo-skills
```

Then add to your Claude Code plugins configuration.

### Per-Project Skills

To install individual skills in a Cairo project:

```bash
git clone https://github.com/feltroidprime/cairo-skills.git ~/.cairo-skills
mkdir -p .claude/skills
ln -s ~/.cairo-skills/skills/benchmarking-cairo .claude/skills/benchmarking-cairo
ln -s ~/.cairo-skills/skills/using-bounded-int .claude/skills/using-bounded-int
```

To update:

```bash
cd ~/.cairo-skills && git pull
```

## What are Claude Code Skills?

Skills are reference guides that help Claude Code apply proven techniques and tool workflows. When a skill is installed (symlinked into `.claude/skills/`), Claude automatically discovers and uses it when the situation matches.

## Contributing

PRs welcome. Each skill is a directory in `skills/` containing at minimum a `SKILL.md` with YAML frontmatter:

```yaml
---
name: skill-name
description: Use when [specific triggering conditions]
---
```

See [skills/benchmarking-cairo/SKILL.md](skills/benchmarking-cairo/SKILL.md) for an example.
