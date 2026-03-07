# Reference Rules (Fork Feature)

> **Note:** This is a feature added in the [hirossyi73/rulesync](https://github.com/hirossyi73/rulesync) fork. It is not available in the upstream rulesync.

## Overview

When managing many rules with rulesync, all generated rule files are auto-loaded by AI tools (e.g., Claude Code) at session start. As the total rule volume grows, this can cause "Prompt is too long" errors or consume excessive context window space.

**Reference rules** solve this problem. By marking a rule with `reference: true`, the generated file is placed in a separate directory that is **not auto-loaded**. Subagents and workflows can still read these files on demand using the `Read` tool.

## How It Works

### Frontmatter

Add `reference: true` to the frontmatter of any `.rulesync/rules/*.md` file:

```md
---
root: false
reference: true
targets: ['*']
description: 'Detailed security guidelines (OWASP Top 10, secret management)'
globs: ['**/*']
---

# Security Guidelines

...
```

### Generated Output

| Tool | Normal Rule Output | Reference Rule Output |
|---|---|---|
| Claude Code | `.claude/rules/<name>.md` | `.claude/references/<name>.md` |
| Other tools (agentsmd, copilot, etc.) | Normal behavior | Excluded from generated output |

For Claude Code specifically:

- **Normal rules** are generated with `paths` frontmatter, causing Claude Code to auto-load them when matching files are accessed.
- **Reference rules** are generated **without `paths` frontmatter** and placed in `.claude/references/`. Claude Code does not auto-load these files.

### AGENTS.md / Copilot

Reference rules are excluded from the references section in `AGENTS.md` and Copilot instructions. They are intended to be accessed directly by file path when needed.

## Use Cases

### Large Rule Sets

If your project has many detailed rules (e.g., security checklists, design standards, coding conventions), marking verbose reference-style rules as `reference: true` keeps the auto-loaded context lean while preserving access.

### Template / Base Projects

For projects that serve as a template (where `.rulesync/` source files are not deployed to target environments), reference rules ensure that detailed guidelines are included in the generated output and available at the deployment target via `.claude/references/`.

## Usage in Subagents and Skills

Reference rules can be read by subagents using the `Read` tool. In your rule or skill files, you can instruct agents to consult reference files:

```md
For detailed security review criteria, read `.claude/references/security.md`.
```

## Which Rules Should Be References?

| Rule Type | `reference: true`? | Reason |
|---|---|---|
| Core conventions (naming, git workflow) | No | Needed in every session |
| Error handling patterns | No | Frequently referenced during coding |
| Detailed security checklists (OWASP) | **Yes** | Long, only needed during security review |
| Detailed design standards | **Yes** | Long, only needed during planning/review |
| Implementation plan templates | **Yes** | Only needed by planner agents |
| Requirements standards | **Yes** | Only needed by planner agents |

The goal is to keep **always-loaded rules under the context limit** while making detailed references available on demand.

## Frontmatter Reference

| Field | Type | Default | Description |
|---|---|---|---|
| `reference` | `boolean` | `false` | When `true`, the rule is output as a reference file (not auto-loaded) |

This field can be combined with other frontmatter fields:

```md
---
root: false
reference: true
targets: ['claudecode']
description: 'Detailed implementation plan standards'
globs: ['**/*']
---
```

> **Note:** `reference: true` with `root: true` is not a meaningful combination. Root rules are already output without `paths` frontmatter. If both are set, `root` takes precedence for path resolution.
