# Claude Code Subagents Guide

> Understanding when and how to use subagents in Claude Code for efficient, context-aware task delegation.

---

## Table of Contents

1. [What Are Subagents?](#what-are-subagents)
2. [Main Agent vs Subagents](#main-agent-vs-subagents)
3. [Built-in Subagents](#built-in-subagents)
4. [When to Use Subagents](#when-to-use-subagents)
5. [How to Trigger Subagents](#how-to-trigger-subagents)
6. [Resuming Subagents](#resuming-subagents)
7. [Background Execution](#background-execution)
8. [Custom Subagents](#custom-subagents)
9. [Common Mistakes](#common-mistakes)
10. [Quick Reference](#quick-reference)

---

## What Are Subagents?

A **subagent** is a separate Claude instance that the main agent spawns to handle specific tasks. Think of it as delegating work to a specialist.

**Key characteristics:**
- Operates in its own **isolated context window** (separate memory)
- Has a **specific purpose** (exploring, planning, reviewing)
- Can have **restricted tool access** (read-only, specific tools only)
- Returns results to the main agent when done
- Runs in the **same terminal** (no new tabs/windows)

**What you experience:**
```
You: "Where is authentication handled in this codebase?"

[Main agent delegates to Explore subagent]
⏳ Running agent: Exploring codebase...

[Results appear inline]
Claude: "Authentication is handled in src/auth/... [summarized findings]"
```

---

## Main Agent vs Subagents

| Aspect | Main Agent | Subagent |
|--------|------------|----------|
| **Context** | Full conversation history | Fresh, isolated context |
| **Scope** | Your entire session | Single delegated task |
| **User interaction** | Direct | Through main agent only |
| **Tools** | All available | Can be restricted |
| **Lifespan** | Entire session | Task completion |
| **Can spawn agents** | Yes | No (no nesting) |

**Important:** You cannot talk to subagents directly. The main agent orchestrates everything:

```
You ←→ Main Agent ←→ Subagent(s)
         ↑
    (orchestrator)
```

---

## Built-in Subagents

Claude Code includes three built-in subagents:

### Explore

| Property | Value |
|----------|-------|
| **Model** | Haiku (fast) |
| **Mode** | Read-only |
| **Tools** | Glob, Grep, Read, Bash (read-only commands) |
| **Purpose** | Fast codebase exploration and search |

**Use cases:**
- Finding where something is implemented
- Understanding codebase structure
- Locating specific functions/classes
- Mapping dependencies

**Thoroughness levels:** Quick, Medium, Very thorough

---

### Plan

| Property | Value |
|----------|-------|
| **Model** | Sonnet |
| **Mode** | Research only |
| **Tools** | Read, Glob, Grep, Bash |
| **Purpose** | Architectural planning and research |

**Use cases:**
- Planning refactors before implementation
- Understanding implementation requirements
- Researching approaches before coding

---

### General-purpose

| Property | Value |
|----------|-------|
| **Model** | Sonnet |
| **Mode** | Full access |
| **Tools** | All tools |
| **Purpose** | Complex multi-step tasks |

**Use cases:**
- Tasks requiring both exploration AND modification
- Multi-file changes with research
- Complex debugging across files

---

## When to Use Subagents

### Use Subagents When:

| Scenario | Why Subagent Helps |
|----------|-------------------|
| **Exploratory research** | Keeps search noise out of main context |
| **Large codebase navigation** | Context efficiency for long sessions |
| **Specialized tasks** | Focused expertise (review, debug, test) |
| **Parallel investigations** | Multiple subagents can run simultaneously |
| **Security-sensitive ops** | Restricted tool access |

### Use Main Agent When:

| Scenario | Why Main Agent Is Better |
|----------|-------------------------|
| **Quick, simple tasks** | No context isolation needed |
| **Tight iteration** | Rapid back-and-forth with you |
| **Results need persistence** | Keep findings in conversation |
| **Specific file reads** | Direct Read tool is faster |
| **Known file locations** | No exploration needed |

### Decision Flowchart

```
Is this a simple, specific task?
├── Yes → Use main agent directly
└── No ↓

Do I know exactly where to look?
├── Yes → Use main agent with Read/Grep
└── No ↓

Is this exploratory/research-heavy?
├── Yes → Use Explore subagent
└── No ↓

Does this need planning before implementation?
├── Yes → Use Plan subagent
└── No ↓

Is this a complex multi-step task?
├── Yes → Use General-purpose subagent
└── No → Use main agent
```

---

## How to Trigger Subagents

### Automatic Delegation

Claude automatically delegates based on:
- Task description matching subagent expertise
- Current context state (large context → more likely to delegate)
- Tool requirements of the task

**Example triggers for automatic delegation:**
```
"Where is error handling implemented?"     → Explore
"Find all API endpoints"                   → Explore
"Plan how to refactor the auth system"     → Plan
"Investigate and fix these failing tests"  → General-purpose
```

### Explicit Requests

You can request specific subagents:

```
"Use the explore subagent to find all database queries"

"Have the plan subagent research how caching works here"

"Ask the general-purpose subagent to find and fix the memory leak"
```

### Specifying Thoroughness (Explore)

```
"Do a quick search for login functions"           → Quick
"Find authentication handling"                     → Medium (default)
"Thoroughly map out the entire auth system"        → Very thorough
```

---

## Resuming Subagents

Each subagent execution gets a unique `agentId`. You can resume previous work:

### How It Works

```
1. Initial request:
   You: "Use explore to analyze the payment module"
   [Subagent completes, returns agentId: abc123]

2. Resume later:
   You: "Resume agent abc123 and now look at refund handling"
   [Continues with full previous context]
```

### Use Cases for Resuming

| Scenario | Benefit |
|----------|---------|
| **Long research sessions** | Break into multiple interactions |
| **Iterative refinement** | Build on previous findings |
| **Multi-step workflows** | Maintain context across steps |
| **Follow-up questions** | Don't restart from scratch |

### Tips

- Claude displays the `agentId` when a subagent completes
- Note the ID if you might want to continue later
- Resumed agents have full context from previous work

---

## Background Execution

Subagents can run in background while you continue working:

### Foreground (Default)
- You see progress in real-time
- Results appear when complete
- Blocks main conversation

### Background
- Subagent works independently
- You continue other tasks
- Check results when ready

```
"Run the test-runner subagent in background while I work on docs"

[Continue working...]

"What did the test-runner find?"
```

### Multiple Parallel Subagents

```
"In parallel:
- Have explore find all API endpoints
- Have the reviewer check my recent changes"

[Both run simultaneously, results return separately]
```

---

## Custom Subagents

You can create specialized subagents for your workflow. Brief overview here—see the **Custom Agents Guide** for full details.

### Quick Setup

```bash
/agents  # Interactive management
```

### File Locations

| Type | Location | Scope |
|------|----------|-------|
| Project | `.claude/agents/` | Current project |
| User | `~/.claude/agents/` | All projects |

### Basic Format

```markdown
---
name: your-agent-name
description: When this agent should be invoked
tools: Read, Grep, Glob, Bash
model: sonnet
---

Your agent's system prompt here.
Include specific instructions and constraints.
```

### Example: Test Runner

```markdown
---
name: test-runner
description: Run tests and fix failures. Use proactively after code changes.
tools: Read, Edit, Bash, Grep
---

You are a test automation expert.

When invoked:
1. Identify relevant test files
2. Run appropriate test commands
3. Analyze failures
4. Propose or implement fixes
5. Verify fixes work
```

---

## Common Mistakes

### 1. Overusing Subagents for Simple Tasks

❌ "Use a subagent to read package.json"
✅ Just ask to read package.json directly

**Why:** Subagents have startup latency. Simple reads are faster direct.

---

### 2. Expecting Direct Subagent Communication

❌ Trying to give follow-up instructions to a running subagent
✅ Let it complete, then resume or start fresh

**Why:** You talk to the main agent only. Subagents work autonomously.

---

### 3. Forgetting Context Isolation

❌ "The subagent found X, now in my main conversation reference X"
✅ Ask Claude to incorporate the subagent's findings explicitly

**Why:** Subagent context is separate. Main agent gets summary, not full context.

---

### 4. Trying to Nest Subagents

❌ Creating a subagent that spawns other subagents
✅ Design flat subagent architecture

**Why:** Subagents cannot spawn other subagents (prevents infinite recursion).

---

### 5. Not Specifying Thoroughness

❌ "Find the auth code" (ambiguous scope)
✅ "Do a thorough search for all authentication-related code"

**Why:** Explicit thoroughness helps Explore subagent calibrate effort.

---

### 6. Ignoring Resume for Long Tasks

❌ Starting fresh each time for ongoing research
✅ Note agentId and resume to maintain context

**Why:** Resuming preserves discoveries and avoids duplicate work.

---

## Quick Reference

### Triggering Subagents

| Intent | Example Prompt |
|--------|---------------|
| Quick search | "Quickly find where X is defined" |
| Deep exploration | "Thoroughly explore how the auth system works" |
| Planning | "Plan how to implement feature X" |
| Complex task | "Find and fix all TypeScript errors" |
| Explicit subagent | "Use the explore subagent to..." |
| Resume | "Resume agent [id] and continue..." |
| Background | "Run [task] in background" |

### Built-in Subagents Cheat Sheet

| Subagent | Model | Mode | Best For |
|----------|-------|------|----------|
| Explore | Haiku | Read-only | Fast search, codebase mapping |
| Plan | Sonnet | Research | Architecture, planning |
| General-purpose | Sonnet | Full | Complex multi-step tasks |

### Performance Tips

| Tip | Impact |
|-----|--------|
| Use Explore for search tasks | Faster (Haiku) + context efficient |
| Specify thoroughness level | Calibrates search effort |
| Resume long-running work | Preserves context |
| Run independent tasks in parallel | Time savings |
| Use main agent for simple tasks | Avoids subagent overhead |

### Context Efficiency

```
Main conversation context:
├── Your messages
├── Claude's responses
├── Tool results (can get large!)
└── Subagent summaries (compact!)
         ↑
    Subagent does heavy lifting,
    returns condensed results
```

---

*Subagents help you work efficiently on complex tasks while keeping your main conversation focused and context-efficient.*
