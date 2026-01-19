# Reference Guides

Personal collection of reference documents for common questions and workflows.

## Structure

Guides are organized by topic:

```
md-guides/
├── git/                    # Git commands and workflows
├── ai/                     # Guides on working with AI
│   └── best-practices/     # Tech-specific rules for AI agents
├── javascript/             # JavaScript and React development
├── vscode/                 # VS Code editor shortcuts and configuration
├── config/                 # System configuration and dotfiles
└── ...                     # More topics as needed
```

## Guides

### Git
- [Pull vs Rebase](git/pull-vs-rebase.md) - Understanding `git pull` strategies (merge, rebase, ff-only)
- [GitHub CLI](git/GITHUB-CLI.md) - Master the `gh` command for GitHub from terminal

### AI
- [AI Planning Conversations](ai/AI-PLANNING-CONVERSATIONS.md) - How to plan projects with AI assistants
- [UI Design with AI](ai/UI-DESIGN-WITH-AI.md) - Using AI for UI/UX design workflows
- [Claude Code Subagents](ai/CLAUDE-CODE-SUBAGENTS.md) - Working with Claude Code's subagent system
- [Claude Code Custom Agents](ai/CLAUDE-CODE-CUSTOM-AGENTS.md) - Creating custom agents in Claude Code
- [Claude Code Custom Commands](ai/CLAUDE-CODE-CUSTOM-COMMANDS.md) - Defining custom slash commands
- [AI Phased Development Guide](ai/AI-PHASED-DEVELOPMENT-GUIDE.md) - Breaking projects into phases with AI

### AI Best Practices (For AI Agents)
Guides that AI agents should follow when working on projects with specific tech stacks:
- [Unity](ai/best-practices/unity.md) - Rules for AI working on Unity projects (ScriptableObjects, meta files, etc.)

### JavaScript
- [Event Handling: preventDefault vs stopPropagation](javascript/event-handling-prevent-default-vs-stop-propagation.md) - Understanding event control methods in JavaScript/React

### VS Code
- [Keyboard Shortcuts](vscode/SHORTCUTS.md) - Essential VS Code keyboard shortcuts cheatsheet

### Config
- [Dotfiles Bare Repo](config/DOTFILES-BARE-REPO.md) - Manage dotfiles with a bare git repository
