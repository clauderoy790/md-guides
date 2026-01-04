# Claude Code Custom Agents Guide

> How to create, configure, and manage custom agents using the `/agents` command.

---

## Table of Contents

1. [What Are Custom Agents?](#what-are-custom-agents)
2. [Quick Start with /agents](#quick-start-with-agents)
3. [File Locations](#file-locations)
4. [Configuration Format](#configuration-format)
5. [Configuration Fields](#configuration-fields)
6. [Available Tools](#available-tools)
7. [Model Options](#model-options)
8. [Practical Examples](#practical-examples)
9. [CLI Configuration](#cli-configuration)
10. [Team Sharing](#team-sharing)
11. [Best Practices](#best-practices)
12. [Quick Reference](#quick-reference)

---

## What Are Custom Agents?

Custom agents are **specialized subagents you define** for your specific workflows. Instead of relying only on built-in agents (Explore, Plan, General-purpose), you create agents tailored to your needs.

**Why create custom agents?**
- Automate repetitive specialized tasks
- Enforce consistent patterns (code review checklist, test procedures)
- Restrict tool access for safety
- Share expertise across your team via git

**Example:** A Unity developer might create:
- `unity-reviewer` - Reviews C# code for Unity best practices
- `playtest-debugger` - Investigates runtime issues from player logs
- `shader-helper` - Assists with shader code and optimization

---

## Quick Start with /agents

The easiest way to manage agents is the interactive command:

```
/agents
```

This opens a menu where you can:
- **View** all available agents (built-in + custom)
- **Create** new agents with guided setup
- **Edit** existing custom agents
- **Delete** custom agents
- **See tool access** for each agent

### Creating Your First Agent

```
/agents
→ Select "Create New Agent"
→ Choose scope: Project or User level
→ Describe what the agent should do
→ Select tools it needs
→ (Optional) Let Claude generate the system prompt
→ Save
```

The agent is immediately available for use.

---

## File Locations

Custom agents are Markdown files stored in specific directories:

| Type | Location | Scope | Priority |
|------|----------|-------|----------|
| **Project** | `.claude/agents/` | Current project only | Highest |
| **User** | `~/.claude/agents/` | All your projects | Lower |

### When to Use Which

| Use Project Level | Use User Level |
|-------------------|----------------|
| Project-specific workflows | Personal productivity agents |
| Team-shared agents (via git) | Cross-project utilities |
| Tech-stack specific (Unity, React) | General helpers |

### Priority Rules

If two agents have the same name:
1. Project-level wins over user-level
2. This lets you override user agents for specific projects

---

## Configuration Format

Each agent is a Markdown file with **YAML frontmatter** + **system prompt**:

```markdown
---
name: agent-name
description: When this agent should be used
tools: Tool1, Tool2, Tool3
model: sonnet
---

System prompt content goes here.

This can be multiple paragraphs with detailed instructions,
examples, constraints, and anything else the agent needs.
```

### File Naming

- Filename: `agent-name.md` (matches the `name` field)
- Use lowercase with hyphens
- Examples: `code-reviewer.md`, `unity-helper.md`, `test-runner.md`

---

## Configuration Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Unique identifier (lowercase, hyphens) |
| `description` | Yes | When Claude should use this agent |
| `tools` | No | Comma-separated tools. Omit = inherit all |
| `model` | No | `haiku`, `sonnet`, `opus`, or `inherit` |
| `permissionMode` | No | How to handle permissions |
| `skills` | No | Skills to auto-load |

### name

```yaml
name: unity-reviewer
```

- Lowercase letters and hyphens only
- Must be unique within scope
- Used to invoke: "Use the unity-reviewer agent"

### description

```yaml
description: Reviews Unity C# code for best practices, performance issues, and common mistakes. Use proactively after writing MonoBehaviour code.
```

- Natural language description
- **Critical for auto-delegation** - Claude uses this to decide when to invoke
- Include "use proactively" to encourage automatic use
- Be specific about the expertise area

### tools

```yaml
tools: Read, Grep, Glob, Bash
```

- Comma-separated list
- **Omit entirely** to inherit all tools (including MCP tools)
- Restrict for security or focus

### model

```yaml
model: sonnet
```

| Value | Use Case |
|-------|----------|
| `haiku` | Fast, simple tasks (search, quick checks) |
| `sonnet` | Balanced (most agents) |
| `opus` | Complex reasoning |
| `inherit` | Use main conversation's model |

### permissionMode

```yaml
permissionMode: default
```

| Mode | Behavior |
|------|----------|
| `default` | Normal permission prompts |
| `acceptEdits` | Auto-accept file edits |
| `dontAsk` | Minimal prompts |
| `bypassPermissions` | Skip all (use carefully) |
| `plan` | Research mode only |

### skills

```yaml
skills: unity-patterns, csharp-standards
```

- Skills to auto-load for this agent
- Skills are NOT inherited from parent by default

---

## Available Tools

Tools you can grant to agents:

| Tool | Purpose |
|------|---------|
| `Read` | Read file contents |
| `Write` | Create/overwrite files |
| `Edit` | Make targeted edits |
| `Glob` | Find files by pattern |
| `Grep` | Search file contents |
| `Bash` | Execute shell commands |
| `WebFetch` | Fetch URL content |
| `WebSearch` | Web search |
| `Task` | Spawn sub-agents (main agent only) |
| `TodoWrite` | Manage task lists |
| `NotebookEdit` | Edit Jupyter notebooks |
| `AskUserQuestion` | Ask user for input |

### Common Tool Combinations

| Agent Type | Suggested Tools |
|------------|-----------------|
| Read-only reviewer | `Read, Grep, Glob, Bash` |
| Code fixer | `Read, Edit, Grep, Glob, Bash` |
| Full access | Omit field (inherit all) |
| Research only | `Read, Grep, Glob, WebSearch` |

---

## Model Options

| Model | Speed | Cost | Best For |
|-------|-------|------|----------|
| `haiku` | Fastest | Lowest | Quick searches, simple checks |
| `sonnet` | Balanced | Medium | Most agents (default) |
| `opus` | Slower | Highest | Complex reasoning, nuanced tasks |
| `inherit` | Varies | Varies | Match main conversation |

**Recommendation:** Start with `sonnet`, use `haiku` for speed-critical agents, `opus` only when you need deeper reasoning.

---

## Practical Examples

### Unity Code Reviewer

```markdown
---
name: unity-reviewer
description: Reviews Unity C# code for best practices, performance, and common pitfalls. Use proactively after writing or modifying MonoBehaviour scripts.
tools: Read, Grep, Glob, Bash
model: sonnet
---

You are a senior Unity developer reviewing C# code for quality and performance.

## When Invoked

1. Run `git diff` to see recent changes
2. Focus on modified `.cs` files
3. Begin review immediately

## Review Checklist

### Unity-Specific
- [ ] No `Find()` or `GetComponent()` in Update/FixedUpdate
- [ ] Coroutines properly stopped on disable/destroy
- [ ] Object pooling used for frequent instantiation
- [ ] No string concatenation in hot paths
- [ ] Proper use of SerializeField vs public
- [ ] Events unsubscribed in OnDisable/OnDestroy
- [ ] No Camera.main in Update (cache it)
- [ ] Appropriate use of CompareTag() vs tag ==

### Performance
- [ ] No allocations in Update loops
- [ ] Physics queries use non-alloc versions
- [ ] Appropriate FixedUpdate vs Update usage
- [ ] No excessive Debug.Log in builds

### Code Quality
- [ ] Follows C# naming conventions
- [ ] No magic numbers (use constants/SerializeField)
- [ ] Null checks where appropriate
- [ ] Clear separation of concerns

## Output Format

Organize feedback by priority:
1. **Critical** - Must fix (performance killers, bugs)
2. **Warning** - Should fix (bad practices)
3. **Suggestion** - Consider improving

Include code examples showing the fix.
```

---

### Unity Playtest Debugger

```markdown
---
name: playtest-debugger
description: Investigates Unity runtime issues from error logs, player reports, or unexpected behavior. Use when debugging gameplay issues.
tools: Read, Edit, Grep, Glob, Bash
model: sonnet
---

You are an expert Unity debugger specializing in runtime issues.

## When Invoked

1. Gather error information (logs, stack traces, reproduction steps)
2. Locate relevant code files
3. Form hypotheses
4. Identify root cause
5. Propose minimal fix

## Debugging Approach

### For Null Reference Exceptions
- Check serialized field assignments in Inspector
- Verify Awake/Start execution order
- Look for race conditions with async operations
- Check scene loading state

### For Physics Issues
- Verify Rigidbody settings
- Check FixedUpdate vs Update usage
- Look for transform manipulation conflicts
- Verify layer collision matrix

### For Performance Issues
- Profile allocation patterns
- Check for expensive operations in Update
- Look for unnecessary GetComponent calls
- Verify object pooling usage

### For Build-Only Issues
- Check for #if UNITY_EDITOR guards
- Verify streaming assets paths
- Check platform-specific code
- Look for missing assembly references

## Output

For each issue:
1. Root cause explanation
2. Evidence from code
3. Minimal fix with code example
4. Prevention recommendation
```

---

### Unity Shader Helper

```markdown
---
name: shader-helper
description: Assists with Unity shader development, optimization, and debugging. Use for HLSL/ShaderLab questions and shader performance issues.
tools: Read, Edit, Grep, Glob
model: sonnet
---

You are a Unity graphics programmer specializing in shaders.

## Expertise Areas

- ShaderLab syntax
- HLSL/Cg programming
- Shader Graph to code conversion
- URP/HDRP shader compatibility
- Mobile shader optimization
- Shader debugging techniques

## When Helping With Shaders

### For New Shaders
- Clarify render pipeline (Built-in, URP, HDRP)
- Understand target platforms
- Determine LOD requirements
- Consider batching compatibility

### For Optimization
- Reduce texture samples
- Minimize branching
- Use appropriate precision (half vs float)
- Consider GPU architecture (mobile vs desktop)
- Batch-friendly property usage

### For Debugging
- Check shader compilation errors
- Verify property bindings
- Test with replacement shaders
- Use frame debugger recommendations

## Common Patterns

### Property Optimization
```hlsl
// Mobile: Use half precision
half4 _Color;

// Batch-compatible properties
UNITY_INSTANCING_BUFFER_START(Props)
    UNITY_DEFINE_INSTANCED_PROP(half4, _Color)
UNITY_INSTANCING_BUFFER_END(Props)
```

Provide complete, working shader code when possible.
```

---

### ScriptableObject Architect

```markdown
---
name: so-architect
description: Designs ScriptableObject architectures for Unity projects. Use when planning data structures, game configuration, or event systems.
tools: Read, Write, Edit, Grep, Glob
model: sonnet
---

You are a Unity architect specializing in ScriptableObject patterns.

## Expertise

- Data-driven game design
- ScriptableObject event systems
- Configuration management
- Runtime vs editor data separation
- Asset workflow optimization

## Common Patterns

### Runtime Sets
```csharp
[CreateAssetMenu]
public class RuntimeSet<T> : ScriptableObject
{
    public List<T> Items = new();
    public void Add(T item) => Items.Add(item);
    public void Remove(T item) => Items.Remove(item);
}
```

### Game Events
```csharp
[CreateAssetMenu]
public class GameEvent : ScriptableObject
{
    private List<GameEventListener> listeners = new();
    public void Raise() => listeners.ForEach(l => l.OnEventRaised());
    public void Register(GameEventListener l) => listeners.Add(l);
    public void Unregister(GameEventListener l) => listeners.Remove(l);
}
```

### Variable References
```csharp
[CreateAssetMenu]
public class FloatVariable : ScriptableObject
{
    public float Value;
}

[Serializable]
public class FloatReference
{
    public bool UseConstant = true;
    public float ConstantValue;
    public FloatVariable Variable;
    public float Value => UseConstant ? ConstantValue : Variable.Value;
}
```

When designing architectures:
1. Understand the game's data requirements
2. Identify what changes at runtime vs design time
3. Plan the asset folder structure
4. Consider editor tooling needs
```

---

### General Code Reviewer (Non-Unity)

```markdown
---
name: code-reviewer
description: Reviews code for quality, security, and maintainability. Use after writing or modifying code.
tools: Read, Grep, Glob, Bash
model: sonnet
---

You are a senior developer reviewing code for quality and security.

## When Invoked

1. Run `git diff` to see recent changes
2. Focus on modified files
3. Begin review immediately

## Review Checklist

- [ ] Code is clear and readable
- [ ] Functions/variables well-named
- [ ] No duplicated code
- [ ] Proper error handling
- [ ] No exposed secrets or API keys
- [ ] Input validation where needed
- [ ] Tests cover new functionality
- [ ] No obvious security vulnerabilities

## Output Format

Organize by priority:
1. **Critical** - Must fix
2. **Warning** - Should fix
3. **Suggestion** - Consider improving

Include specific code examples for fixes.
```

---

## CLI Configuration

For scripting or quick testing, define agents via command line:

```bash
claude --agents '{
  "unity-reviewer": {
    "description": "Reviews Unity C# code for best practices",
    "prompt": "You are a Unity code reviewer. Check for performance issues, proper component usage, and C# best practices.",
    "tools": ["Read", "Grep", "Glob", "Bash"],
    "model": "sonnet"
  }
}'
```

### CLI Format

| Field | Required | Description |
|-------|----------|-------------|
| `description` | Yes | When to invoke |
| `prompt` | Yes | System prompt |
| `tools` | No | Array of tools |
| `model` | No | Model alias |

### Use Cases

- Quick testing before committing to file
- Automation scripts
- Sharing agent definitions in documentation
- Session-specific agents

---

## Team Sharing

### Via Git (Recommended)

```
my-unity-project/
├── .claude/
│   └── agents/
│       ├── unity-reviewer.md
│       ├── playtest-debugger.md
│       └── shader-helper.md
├── Assets/
└── README.md
```

1. Create agents in `.claude/agents/`
2. Commit to git
3. Team members get agents automatically

### Benefits

- Version controlled
- Consistent across team
- Evolves with project
- Easy onboarding

---

## Best Practices

### 1. Write Specific Descriptions

❌ `description: Helps with code`
✅ `description: Reviews Unity C# code for performance issues and best practices. Use proactively after modifying MonoBehaviour scripts.`

**Why:** Claude uses descriptions for auto-delegation. Specific = better matching.

---

### 2. Include "Use Proactively"

```yaml
description: ... Use proactively after writing code.
```

**Why:** Encourages Claude to invoke automatically without explicit request.

---

### 3. Restrict Tools Appropriately

| Agent Purpose | Tools to Grant |
|---------------|----------------|
| Reviewer | `Read, Grep, Glob, Bash` (no Edit/Write) |
| Fixer | Add `Edit` |
| Generator | Add `Write` |

**Why:** Limits scope, prevents accidents, focuses the agent.

---

### 4. Structure System Prompts

```markdown
## Role
Who the agent is

## When Invoked
First steps to take

## Checklist/Process
What to check or do

## Output Format
How to present results
```

**Why:** Clear structure = consistent behavior.

---

### 5. Start Simple, Iterate

1. Create minimal agent
2. Use it
3. Notice what's missing
4. Improve the prompt
5. Repeat

**Why:** Hard to predict perfect prompt upfront.

---

### 6. Use Project Level for Team Agents

Keep team-shared agents in `.claude/agents/` (project level), personal helpers in `~/.claude/agents/` (user level).

---

## Quick Reference

### Creating an Agent

**Interactive:**
```
/agents → Create New Agent
```

**Manual:**
```bash
mkdir -p .claude/agents
# Create .claude/agents/my-agent.md with format below
```

### Agent File Template

```markdown
---
name: my-agent
description: What this agent does. Use proactively when [condition].
tools: Read, Grep, Glob, Bash
model: sonnet
---

You are a [role] specializing in [expertise].

## When Invoked

1. First step
2. Second step
3. Third step

## Checklist

- [ ] Check item 1
- [ ] Check item 2
- [ ] Check item 3

## Output Format

Organize results as:
1. Critical issues
2. Warnings
3. Suggestions
```

### Invoking Agents

```
"Use the unity-reviewer agent to check my changes"
"Have the debugger investigate this error"
"Ask the shader-helper about mobile optimization"
```

### Managing Agents

```
/agents              # View and manage all agents
/agents              # Edit existing agents
/agents              # Delete agents
```

---

*Custom agents let you encode your expertise and workflows into reusable, shareable helpers.*
