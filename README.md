# Cairo Skills for Claude Code

Skills for profiling and developing Cairo programs with [Claude Code](https://docs.anthropic.com/en/docs/claude-code).

## Available Skills

| Skill | Description |
|-------|-------------|
| [benchmarking-cairo](benchmarking-cairo/) | Profile Cairo functions with cairo-profiler and pprof |

## Installation

From your Cairo project root:

```bash
git clone https://github.com/feltroidprime/cairo-skills.git ~/.cairo-skills
mkdir -p .claude/skills
ln -s ~/.cairo-skills/benchmarking-cairo .claude/skills/benchmarking-cairo
```

To update:

```bash
cd ~/.cairo-skills && git pull
```

## What are Claude Code Skills?

Skills are reference guides that help Claude Code apply proven techniques and tool workflows. When a skill is installed (symlinked into `.claude/skills/`), Claude automatically discovers and uses it when the situation matches.

## Contributing

PRs welcome. Each skill is a directory containing at minimum a `SKILL.md` with YAML frontmatter:

```yaml
---
name: skill-name
description: Use when [specific triggering conditions]
---
```

See [benchmarking-cairo/SKILL.md](benchmarking-cairo/SKILL.md) for an example.
