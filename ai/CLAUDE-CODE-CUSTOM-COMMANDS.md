# Claude Code Custom Commands Guide

> How to create reusable slash commands for frequently used prompts and workflows.

---

## Table of Contents

1. [What Are Custom Commands?](#what-are-custom-commands)
2. [Commands vs Agents](#commands-vs-agents)
3. [Quick Start](#quick-start)
4. [File Locations](#file-locations)
5. [Command Format](#command-format)
6. [Arguments and Placeholders](#arguments-and-placeholders)
7. [Dynamic Content](#dynamic-content)
8. [Frontmatter Options](#frontmatter-options)
9. [Practical Examples](#practical-examples)
10. [Organizing Commands](#organizing-commands)
11. [Best Practices](#best-practices)
12. [Quick Reference](#quick-reference)

---

## What Are Custom Commands?

Custom commands are **reusable prompt templates** you invoke with a slash. Instead of typing the same instructions repeatedly, you save them as a command.

**Example:**

Instead of typing:
```
Review this code for Unity best practices, check for performance issues
in Update loops, verify proper component caching, and ensure events
are properly unsubscribed...
```

You create `/unity-review` and just type:
```
/unity-review
```

**Key characteristics:**
- Triggered explicitly with `/command-name`
- Injects prompt into your current conversation
- Can include file contents, bash output, arguments
- Stays in the same conversation context
- Simple Markdown files

---

## Commands vs Agents

| Aspect | Commands | Agents |
|--------|----------|--------|
| **What it does** | Injects a prompt | Spawns separate AI instance |
| **Invocation** | Explicit only (`/name`) | Explicit or automatic |
| **Context** | Same conversation | Isolated context window |
| **Complexity** | Simple prompt text | Full system prompt + config |
| **File format** | Single `.md` file | `.md` with YAML frontmatter |
| **Use case** | Shortcuts, templates | Specialized expertise |

### When to Use Which

**Use Commands when:**
- You have a prompt you type repeatedly
- You want a quick shortcut
- The task runs in your current conversation
- You need to pass arguments easily

**Use Agents when:**
- Task needs specialized expertise/persona
- You want automatic delegation
- Task benefits from isolated context
- You need to restrict tools

**Example comparison:**

```
# Command: Quick prompt injection
/unity-review
→ Injects review prompt into current conversation
→ Claude reviews in same context

# Agent: Specialized worker
"Use the unity-reviewer agent"
→ Spawns separate context
→ Agent works independently
→ Returns summary to main conversation
```

---

## Quick Start

### Create Your First Command

```bash
# Create commands directory
mkdir -p .claude/commands

# Create a simple command
echo "Review this code for performance issues and suggest optimizations." > .claude/commands/perf-review.md
```

### Use It

```
/perf-review
```

That's it! The prompt in the file gets injected.

---

## File Locations

| Type | Location | Scope |
|------|----------|-------|
| **Project** | `.claude/commands/` | Current project |
| **User** | `~/.claude/commands/` | All projects |

### Subdirectories for Namespacing

```
.claude/commands/
├── review.md                    # /review
├── unity/
│   ├── check-update.md         # /check-update (project:unity)
│   └── profile.md              # /profile (project:unity)
└── git/
    └── squash.md               # /squash (project:git)
```

Commands in subdirectories show their namespace in `/help`:
- `/review` (project)
- `/check-update` (project:unity)
- `/squash` (project:git)

---

## Command Format

Commands are Markdown files with optional YAML frontmatter:

### Simple Command (No Frontmatter)

```markdown
Review this code for:
- Performance issues
- Security vulnerabilities
- Code style violations

Provide specific line numbers and suggested fixes.
```

### Command with Frontmatter

```markdown
---
description: Review code for Unity best practices
argument-hint: [file-path]
---

Review the Unity C# code for:
- Update loop performance
- Component caching
- Event subscription cleanup
- SerializeField usage

Focus on file: $1
```

---

## Arguments and Placeholders

Commands can accept arguments when invoked.

### Available Placeholders

| Placeholder | Description |
|-------------|-------------|
| `$ARGUMENTS` | All arguments as one string |
| `$1` | First argument |
| `$2` | Second argument |
| `$3` | Third argument |
| ... | Up to `$9` |

### Example: Single Argument

**Command file:** `.claude/commands/explain.md`
```markdown
---
argument-hint: [file-path]
---

Explain what this file does in simple terms: $1
```

**Usage:**
```
/explain src/PlayerController.cs
```

### Example: Multiple Arguments

**Command file:** `.claude/commands/compare.md`
```markdown
---
argument-hint: [file1] [file2]
---

Compare these two files and explain the differences:
- File 1: $1
- File 2: $2

Focus on functional differences, not just syntax.
```

**Usage:**
```
/compare src/old/Player.cs src/new/Player.cs
```

### Example: All Arguments

**Command file:** `.claude/commands/review-files.md`
```markdown
---
argument-hint: [files...]
---

Review all these files for consistency: $ARGUMENTS
```

**Usage:**
```
/review-files src/Player.cs src/Enemy.cs src/NPC.cs
```

---

## Dynamic Content

Commands can include dynamic content that gets evaluated at runtime.

### Include File Contents with @

```markdown
Review this configuration file for issues:

@config/settings.json

Check for:
- Missing required fields
- Invalid values
- Security concerns
```

The contents of `config/settings.json` get inserted into the prompt.

### Include Bash Output with !

```markdown
Current git status:
!`git status`

Recent commits:
!`git log --oneline -5`

Based on the above, suggest what to commit next.
```

The bash commands run and their output gets inserted.

### Combining Dynamic Content

```markdown
---
description: Prepare a commit with context
---

## Current State

Branch: !`git branch --show-current`

Status:
!`git status --short`

Diff:
!`git diff --cached`

## Task

Based on the staged changes above, create a meaningful commit message.
```

---

## Frontmatter Options

| Field | Purpose | Example |
|-------|---------|---------|
| `description` | Shows in `/help` | `description: Review Unity code` |
| `argument-hint` | Shows expected args | `argument-hint: [file] [line]` |
| `allowed-tools` | Restrict tool access | `allowed-tools: Bash(git:*)` |
| `model` | Use specific model | `model: claude-opus-4-5-20251101` |
| `disable-model-invocation` | Prevent auto-invoke | `disable-model-invocation: true` |

### allowed-tools Examples

```yaml
# Only allow specific git commands
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git add:*)

# Only allow read operations
allowed-tools: Read, Glob, Grep

# Allow npm commands
allowed-tools: Bash(npm run:*), Bash(npm test:*)
```

---

## Practical Examples

### Unity: Check Update Loop

```markdown
---
description: Find performance issues in Update methods
---

Search for Update(), FixedUpdate(), and LateUpdate() methods in C# files.

For each one found, check for:
- GetComponent calls (should be cached)
- Find() calls (expensive)
- Camera.main access (should be cached)
- String concatenation (causes GC)
- LINQ in hot paths
- Allocations (new, boxing)

Report issues with file paths and line numbers.
```

**Usage:** `/check-update`

---

### Unity: Review MonoBehaviour

```markdown
---
description: Review a MonoBehaviour script
argument-hint: [script-path]
---

Review the MonoBehaviour at $1 for:

## Unity Best Practices
- [ ] Components cached in Awake/Start, not Update
- [ ] Events unsubscribed in OnDisable/OnDestroy
- [ ] Coroutines stopped when disabled
- [ ] No Find() in runtime methods
- [ ] CompareTag() instead of tag ==

## Performance
- [ ] No allocations in Update
- [ ] Physics queries use NonAlloc versions
- [ ] Object pooling for frequent spawns

## Code Quality
- [ ] SerializeField for inspector values
- [ ] No public fields (use properties or SerializeField)
- [ ] Null checks for optional references

Provide specific fixes for any issues found.
```

**Usage:** `/review-mono Assets/Scripts/PlayerController.cs`

---

### Unity: Generate Test Stub

```markdown
---
description: Generate unit test stub for a class
argument-hint: [class-path]
---

Read the class at $1 and generate a unit test file for it.

Requirements:
- Use Unity Test Framework (NUnit)
- Create test for each public method
- Include setup/teardown if needed
- Add comments for what each test should verify
- Use AAA pattern (Arrange, Act, Assert)

Output the complete test file.
```

**Usage:** `/gen-test Assets/Scripts/InventoryManager.cs`

---

### Git: Smart Commit

```markdown
---
description: Create a commit with AI-generated message
allowed-tools: Bash(git:*)
---

## Current State

Status:
!`git status --short`

Staged changes:
!`git diff --cached --stat`

Detailed diff:
!`git diff --cached`

## Task

Based on the staged changes:
1. Summarize what changed
2. Generate a clear commit message following conventional commits
3. Create the commit

Format: `type(scope): description`
Types: feat, fix, refactor, docs, test, chore
```

**Usage:** `/smart-commit`

---

### General: Explain Error

```markdown
---
description: Explain an error message
argument-hint: [error-message]
---

Explain this error in simple terms:

```
$ARGUMENTS
```

Include:
1. What the error means
2. Common causes
3. How to fix it
4. How to prevent it

If it's a stack trace, identify the most likely source of the problem.
```

**Usage:** `/explain-error NullReferenceException: Object reference not set...`

---

### General: Document Function

```markdown
---
description: Add documentation to a function
argument-hint: [file:line] or [function-name]
---

Find and document the function at $1.

Add:
- Summary of what it does
- Parameter descriptions
- Return value description
- Example usage if helpful
- Any important notes about behavior

Use the appropriate doc format for the language:
- C#: XML docs (///)
- JS/TS: JSDoc
- Python: Docstrings
```

**Usage:** `/document-func src/utils.cs:42`

---

### Project: Quick Setup Info

```markdown
---
description: Show project setup commands
---

## Project Setup

To get started with this project:

!`cat README.md | head -50`

## Available Scripts

!`cat package.json | grep -A 20 '"scripts"'`

## Recent Activity

!`git log --oneline -10`
```

**Usage:** `/setup-info`

---

## Organizing Commands

### Recommended Structure

```
.claude/commands/
├── review.md              # General code review
├── explain.md             # Explain code/errors
│
├── unity/                 # Unity-specific
│   ├── check-update.md
│   ├── review-mono.md
│   └── gen-test.md
│
├── git/                   # Git workflows
│   ├── smart-commit.md
│   ├── pr-summary.md
│   └── changelog.md
│
└── docs/                  # Documentation
    ├── document-func.md
    └── gen-readme.md
```

### Personal vs Project Commands

**Project commands** (`.claude/commands/`):
- Shared with team via git
- Project-specific workflows
- Tech stack specific

**Personal commands** (`~/.claude/commands/`):
- Your personal shortcuts
- Cross-project utilities
- Experimental commands

---

## Best Practices

### 1. Write Clear Descriptions

```yaml
# Good
description: Review Unity MonoBehaviour for performance issues

# Bad
description: Review code
```

Descriptions show in `/help` - make them useful.

---

### 2. Use Argument Hints

```yaml
# Helps users know what to provide
argument-hint: [file-path] [optional-line-number]
```

---

### 3. Keep Commands Focused

❌ One command that does review + fix + test + commit

✅ Separate commands:
- `/review` - Review code
- `/fix` - Fix issues
- `/test` - Generate tests
- `/commit` - Create commit

---

### 4. Include Context in Prompts

```markdown
## Context
You are reviewing Unity C# code for a mobile game.

## Constraints
- Must run on low-end Android devices
- Memory budget is tight

## Task
Review this code for performance issues...
```

---

### 5. Test Before Sharing

Run your command a few times before committing. Check:
- Arguments work as expected
- Dynamic content (`@` and `!`) resolves correctly
- Output is useful

---

### 6. Version Control Project Commands

```bash
git add .claude/commands/
git commit -m "Add team code review commands"
```

Team members get your commands automatically.

---

## Quick Reference

### Creating Commands

```bash
# Project command
mkdir -p .claude/commands
echo "Your prompt here" > .claude/commands/my-command.md

# Personal command
mkdir -p ~/.claude/commands
echo "Your prompt here" > ~/.claude/commands/my-command.md
```

### Command Template

```markdown
---
description: What this command does
argument-hint: [arg1] [arg2]
---

Your prompt here.

Use $1 for first argument, $2 for second, etc.
Use $ARGUMENTS for all arguments.
Use @file/path to include file contents.
Use !`command` to include bash output.
```

### Viewing Commands

```
/help    # Lists all commands with descriptions
```

### Invoking Commands

```
/command-name
/command-name arg1 arg2
/command-name "argument with spaces"
```

### Dynamic Content Syntax

| Syntax | Purpose | Example |
|--------|---------|---------|
| `$1`, `$2`... | Positional arguments | `$1` |
| `$ARGUMENTS` | All arguments | `$ARGUMENTS` |
| `@path` | Include file | `@src/main.cs` |
| `!`command`` | Include bash output | `!`git status`` |

### Frontmatter Cheat Sheet

```yaml
---
description: Shows in /help
argument-hint: [required] [optional]
allowed-tools: Read, Grep, Bash(git:*)
model: claude-opus-4-5-20251101
disable-model-invocation: true
---
```

---

*Custom commands turn your repeated prompts into quick shortcuts you can share with your team.*
