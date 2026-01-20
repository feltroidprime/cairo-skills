---
title: Prompt Optimize
description: Maximize signal-to-noise ratio in prompts, skills, and agent instructions
tags: [meta, optimization, prompts, efficiency]
version: 1.0.0
complexity: moderate
---

# Prompt Optimize

Analyze and optimize any prompt, skill, or agent instruction for maximum information density.

## Triggers
- "Optimize this prompt"
- "Make this skill more efficient"
- "Reduce token usage"
- "Improve signal to noise"

## Optimization Principles

### 1. Front-Load Output Format
Put the expected output structure first. Claude reads sequentially—seeing the target shape early focuses all subsequent instructions.

```
BAD:  [context] → [process] → [finally, the format]
GOOD: [output format] → [how to fill it] → [edge cases]
```

### 2. Trust Model Knowledge
Delete explanations of things Claude knows:
- Programming syntax, standard libraries, CLI tools
- Common formats (JSON, YAML, Markdown, BDD)
- Well-known patterns (REST, MVC, CRUD)

### 3. Tables Over Prose
Convert descriptive lists to tables. Encode relationships spatially.

```
BAD:
- The name field is required and should be kebab-case
- The description field is required and should be one line
- The version field is required and uses semver

GOOD:
| Field | Required | Format |
|-------|----------|--------|
| name | yes | kebab-case |
| description | yes | one line |
| version | yes | semver |
```

### 4. Imperative Over Descriptive
Use commands, not descriptions of what should happen.

```
BAD:  "The system should validate the input before processing"
GOOD: "Validate input. Then process."
```

### 5. Remove Meta-Commentary
Delete phrases that describe without instructing:
- "You are an AI assistant that..."
- "This skill is designed to..."
- "The purpose of this section is..."
- "It's important to note that..."

### 6. Deduplicate
If information appears twice, delete one instance:
- Examples that just restate the template
- Explanations of fields already shown in structure
- Error handling repeated in multiple sections

### 7. Implicit Over Explicit
Let structure convey meaning. Headers, indentation, and ordering communicate without tokens.

```
BAD:  "First do X, then do Y, finally do Z"
GOOD:
1. X
2. Y
3. Z
```

## Process

1. **Measure**: Estimate current tokens (~4 chars/token)
2. **Identify**: Mark redundancy, prose-heavy sections, buried critical info
3. **Restructure**: Move output format to top
4. **Compress**: Apply principles above
5. **Validate**: Ensure no semantic loss
6. **Measure**: Compare token counts

## Analysis Template

When analyzing a prompt, produce:

```
## Token Analysis
- Current: ~{n} tokens
- Target: ~{n} tokens ({x}% reduction)

## Issues Found
| Location | Issue | Tokens Wasted |
|----------|-------|---------------|
| {line/section} | {problem} | ~{n} |

## Optimized Version
{rewritten prompt}

## Validation
- [ ] Same output capability
- [ ] No lost edge cases
- [ ] Critical info front-loaded
```

## Edge Cases

| Scenario | Action |
|----------|--------|
| Prompt is already minimal | Report "already optimized" with score |
| Removing content changes behavior | Keep it, note tradeoff |
| User wants specific verbosity | Respect preference, optimize within constraint |
