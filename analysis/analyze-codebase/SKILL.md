---
title: Analyze Codebase
description: Comprehensive codebase analysis - architecture, patterns, dependencies, structure, and quality assessment
tags:
  - analysis
  - architecture
  - code-quality
  - documentation
version: 1.0.0
---

# Codebase Analysis Skill

Use this skill to perform a thorough analysis of a codebase. This provides a systematic approach to understanding unfamiliar projects or documenting existing ones.

## When to Use

- Onboarding to a new project
- Creating documentation for a codebase
- Understanding project architecture before making changes
- Reviewing code quality and patterns
- Identifying technical debt or improvement opportunities

## Analysis Framework

### 1. Project Overview

First, gather high-level information:

- **Project type**: Library, CLI tool, web app, API, monorepo, etc.
- **Primary language(s)**: Check file extensions and config files
- **Build system**: Look for package.json, Cargo.toml, pyproject.toml, Makefile, etc.
- **Framework(s)**: Identify core frameworks (React, Django, Spring, etc.)

Key files to examine:
- README.md, CONTRIBUTING.md, AGENTS.md
- Package manifests (package.json, Cargo.toml, go.mod, pyproject.toml)
- Configuration files (tsconfig.json, .eslintrc, pytest.ini)

### 2. Directory Structure Analysis

Map the project layout:

```
Use glob patterns to understand structure:
- **/* for top-level directories
- src/**/* or lib/**/* for source organization
- tests/**/* or *_test.* for test structure
```

Identify:
- Entry points (main.*, index.*, app.*)
- Source vs generated/build directories
- Configuration locations
- Documentation locations

### 3. Architecture Patterns

Look for common patterns:

**Layered Architecture**:
- controllers/, services/, repositories/, models/
- api/, domain/, infrastructure/

**Feature-based**:
- features/<name>/ with collocated files
- modules/<name>/

**Monorepo**:
- packages/*, apps/*, libs/*
- Workspace configuration (pnpm-workspace.yaml, lerna.json)

### 4. Dependency Analysis

**External dependencies**:
- Parse package manifests for direct dependencies
- Identify major frameworks and libraries
- Check for outdated or deprecated packages

**Internal dependencies**:
- Trace import/require statements
- Identify core modules that many files depend on
- Find circular dependency risks

### 5. Code Patterns & Conventions

Analyze coding style:

- **Naming conventions**: camelCase, snake_case, PascalCase
- **File organization**: One class per file, barrel exports, etc.
- **Error handling**: Try/catch patterns, Result types, error boundaries
- **State management**: Redux, Zustand, Context, signals
- **API patterns**: REST, GraphQL, RPC

### 6. Testing Strategy

Identify test approach:

- **Test types**: Unit, integration, e2e, snapshot
- **Test location**: Collocated, separate directory, both
- **Coverage**: Check for coverage configuration
- **Test frameworks**: Jest, pytest, cargo test, go test

### 7. Configuration & Environment

Document configuration:

- Environment variables (.env.example, config files)
- Feature flags
- Build-time vs runtime configuration
- Secrets management approach

### 8. CI/CD & DevOps

Check automation:

- `.github/workflows/`, `.gitlab-ci.yml`, `Jenkinsfile`
- Dockerfile, docker-compose.yml
- Deployment configuration
- Pre-commit hooks (.husky/, .pre-commit-config.yaml)

## Output Format

Structure your analysis report as:

```markdown
# Codebase Analysis: [Project Name]

## Quick Summary
- **Type**: [e.g., TypeScript monorepo, Python web API]
- **Size**: [file count, LOC estimate]
- **Maturity**: [early stage, production, legacy]

## Architecture
[Describe the overall architecture pattern]

## Key Components
| Component | Purpose | Location |
|-----------|---------|----------|
| ... | ... | ... |

## Technology Stack
- **Runtime**: ...
- **Framework**: ...
- **Database**: ...
- **Key Libraries**: ...

## Entry Points
- Main application: `path/to/main`
- API endpoints: `path/to/routes`
- CLI commands: `path/to/commands`

## Code Quality Observations
- [Strengths]
- [Areas for improvement]
- [Technical debt]

## Recommendations
- [Suggested improvements]
- [Documentation gaps]
- [Refactoring opportunities]
```

## Tips

1. **Start broad, then deep**: Get the big picture before diving into specific modules
2. **Follow the data**: Trace how data flows through the system
3. **Check tests first**: Tests often document expected behavior
4. **Read error handling**: How errors propagate reveals architecture
5. **Find the domain**: Look for business logic separate from infrastructure

## Commands to Run

```bash
# Get file counts by extension
find . -type f | grep -E '\.[a-z]+$' | sed 's/.*\.//' | sort | uniq -c | sort -rn

# Find largest files (potential complexity)
find . -type f -name "*.ts" -exec wc -l {} + | sort -rn | head -20

# List all unique import sources
grep -rh "^import\|^from" --include="*.py" | sort -u

# Check git history for hot spots
git log --format=format: --name-only | sort | uniq -c | sort -rn | head -20
```
