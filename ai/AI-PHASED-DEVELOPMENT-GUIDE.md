# AI-Phased Development Guide

> A methodology for breaking complex projects into AI-implementable phases that can be tested incrementally and potentially parallelized across multiple AI agents.

---

## Table of Contents

1. [Quick Start](#quick-start)
2. [Claude Code Integration](#claude-code-integration)
3. [Philosophy](#philosophy)
4. [Document Structure](#document-structure)
5. [Project Setup (CLAUDE.md / GEMINI.md)](#project-setup-claudemd--geminimd)
6. [Writing the Main Plan](#writing-the-main-plan)
7. [Writing Implementation Phases](#writing-implementation-phases)
8. [Self-Contained Phase Prompts](#self-contained-phase-prompts)
9. [Phase Design Principles](#phase-design-principles)
10. [Parallel Execution](#parallel-execution)
11. [Testing Between Phases](#testing-between-phases)
12. [Context Management](#context-management)
13. [Example Workflow](#example-workflow)

---

## Quick Start

Once set up, implementing a phase is as simple as:

```
You: /implement 3
```

Or the classic way:
```
You: "implement phase 3"
```

The agent reads your project's `CLAUDE.md` or `GEMINI.md`, finds your plans, and executes the phase with full context.

**Setup required:**
1. Create `CLAUDE.md` and `GEMINI.md` in project root (same content, points to plans)
2. Create `plans/` directory with main plan + implementation doc
3. Each phase in implementation doc is self-contained

See [Project Setup](#project-setup-claudemd--geminimd) for details.

---

## Claude Code Integration

If using Claude Code, you have access to custom commands, agents, and skills that enhance the phased workflow.

### Quick Commands

| Command | Description |
|---------|-------------|
| `/implement N` | Implement phase N directly |
| `/next` | Find and show the next uncompleted phase |
| `/phase-status` | Show current progress overview |

### Specialized Agents

| Agent | When to Use |
|-------|-------------|
| `phase-reviewer` | After completing a phase - reviews work for quality |
| `test-guide` | After implementation - walks through manual testing |

**Usage:**
```
"Use the phase-reviewer agent to check my work"
"Have the test-guide agent help me test this phase"
```

### Built-in Subagents

| Subagent | When to Use |
|----------|-------------|
| **Explore** | Research codebase before planning phases |
| **Plan** | Design new phases or refine existing ones |

**Usage:**
```
"Use the explore subagent to map out the auth system"
"Use the plan subagent to design Phase 7 for notifications"
```

### Enhanced Workflow

| Step | Command/Action |
|------|----------------|
| Check progress | `/phase-status` |
| Start next phase | `/next` or `/implement N` |
| After implementing | "Use phase-reviewer agent" |
| Before manual testing | "Use test-guide agent" |
| Research codebase | "Use explore subagent to..." |
| Plan new features | "Use plan subagent to..." |

### Installation

These tools are installed at user level (`~/.claude/`) and work across all projects:

```
~/.claude/
├── commands/
│   ├── implement.md
│   ├── next.md
│   └── phase-status.md
├── agents/
│   ├── phase-reviewer.md
│   └── test-guide.md
└── skills/
    └── phased-development/
        └── SKILL.md
```

No per-project setup needed - they're available everywhere.

---

## Philosophy

### Why Phased Development?

Traditional development often leads to:
- Massive PRs that are hard to review
- Features that don't work because too much was built at once
- Wasted AI tokens on context that isn't needed
- Difficulty identifying where bugs originated

**Phased development solves this by:**
- Breaking work into testable increments
- Keeping AI context focused and minimal
- Allowing parallel work on independent features
- Making debugging easier (you know which phase broke things)

### Core Principles

1. **Each phase = One AI conversation** - A phase should be completable in a single Claude/Gemini session without running out of context
2. **Testable in isolation** - After each phase, you can manually verify it works
3. **Minimal dependencies** - Phases should have clear, minimal dependencies on other phases
4. **No YOLO deployments** - Never implement 20 features and hope they work together

---

## Document Structure

For any significant project, create TWO documents:

```
plans/
├── N_project-name.md              # Main Plan (overview, architecture, decisions)
└── N_project-name-implementation.md  # Implementation Phases (step-by-step)
```

### Main Plan (`N_project-name.md`)

High-level document containing:
- Project overview and goals
- Tech stack decisions
- Architecture diagrams
- Database schema
- API design
- Key decisions and trade-offs
- Future considerations

**Purpose**: Understanding WHAT we're building and WHY

### Implementation Plan (`N_project-name-implementation.md`)

Detailed phases containing:
- Sequential/parallel phase breakdown
- Exact steps per phase
- Files to create/modify
- Manual testing checklist per phase
- Dependencies between phases
- AI prompts for each phase

**Purpose**: Understanding HOW to build it, step by step

---

## Project Setup (CLAUDE.md / GEMINI.md)

### The Entry Point

Create agent instruction files in your project root. These tell AI agents how to work on your project.

```
project-root/
├── CLAUDE.md      # Instructions for Claude agents
├── GEMINI.md      # Instructions for Gemini agents (same content)
└── plans/
    ├── 1_project.md
    └── 1_project-implementation.md
```

**Why separate files?**
- Claude reads `CLAUDE.md` automatically
- Gemini reads `GEMINI.md` automatically
- Same content in both, just different filenames for each AI's convention

**Tip:** Keep them identical. Edit one, copy to the other. Or use a symlink:
```bash
ln -s CLAUDE.md GEMINI.md
```

```markdown
# Project Name

> One-line description of what this project is.

## Phased Development

This project uses phased development. Plans are located in:

- **Main Plan:** `plans/1_project-name.md` - Architecture, schema, decisions
- **Implementation:** `plans/1_project-name-implementation.md` - Phase-by-phase guide

### How to Implement a Phase

When asked to "implement phase N":
1. Read the implementation doc above
2. Find "Phase N" section
3. Follow the self-contained prompt for that phase
4. Run the manual tests before marking complete

### Current Status

- [x] Phase 1: Project Setup
- [x] Phase 2: Database Schema
- [ ] Phase 3: Authentication ← Next
- [ ] Phase 4: Core UI Components
- [ ] ...

## Tech Stack

- Frontend: Next.js 14 (App Router)
- Database: Supabase (PostgreSQL)
- Styling: Tailwind CSS
- Hosting: Vercel

## Project-Specific Notes

[Any coding conventions, patterns, or gotchas specific to this project]
```

### Why These Files?

1. **Single entry point** - Agent reads this first, knows where everything is
2. **Status tracking** - See which phases are done at a glance
3. **Onboarding** - New agents (or humans) understand the project immediately
4. **Convention over configuration** - Standardized across all your projects
5. **Multi-agent support** - Claude reads CLAUDE.md, Gemini reads GEMINI.md

### Workflow

With both files set up, your workflow becomes:

```
# Start a new Claude or Gemini agent
You: "implement phase 3"

# Agent automatically:
# 1. Reads CLAUDE.md (or GEMINI.md)
# 2. Finds plans/1_project-name-implementation.md
# 3. Locates Phase 3 section
# 4. Executes the self-contained prompt
# 5. Runs manual tests
```

No copy-pasting prompts. No explaining context. Just "implement phase N".

Works with both Claude and Gemini (or any AI that reads its instruction file).

---

## Writing the Main Plan

### Required Sections

```markdown
# Project Name - Main Plan

## Overview
- What is this project?
- Who is it for?
- What problem does it solve?

## Tech Stack
| Component | Technology | Justification |
|-----------|------------|---------------|
| Frontend  | Next.js    | SSR, API routes |
| Database  | Supabase   | Free tier, realtime |
| ...       | ...        | ...           |

## Architecture
- System diagram (ASCII or linked image)
- Data flow explanation
- Key components and their responsibilities

## Database Schema
- All tables with columns
- Relationships
- Indexes
- RLS policies (if applicable)

## API Design
- Endpoints list
- Request/response formats
- Authentication method

## Key Decisions
- Trade-offs made and why
- Alternatives considered
- Future migration paths

## Out of Scope (v1)
- Features explicitly NOT included
- Future enhancement ideas
```

### Tips for Main Plan

1. **Be decisive** - Don't leave options open. Pick a direction.
2. **Include diagrams** - ASCII diagrams work great in markdown
3. **Document the "why"** - Future you (or AI) needs to understand decisions
4. **Keep it updated** - If plans change, update the document

---

## Writing Implementation Phases

### Phase Template

```markdown
## Phase N: [Descriptive Name]

**Goal:** One sentence describing what this phase achieves

**Prerequisites:**
- Phase X completed
- [External dependency, e.g., "API key obtained"]

**Estimated AI Complexity:** [Simple | Medium | Complex]
- Simple: 1-2 files, straightforward logic
- Medium: 3-5 files, some complexity
- Complex: 6+ files or intricate logic (consider splitting)

**Dependencies:**
- Creates: [files/features this phase creates]
- Requires: [files/features from previous phases]

### Steps

1. **Step name**
   - Detailed instructions
   - Code snippets if helpful
   - Expected outcome

2. **Another step**
   - ...

### Files Created/Modified
- `path/to/file.ts` - Description
- `path/to/another.ts` - Description

### Manual Testing Checklist
- [ ] Test case 1 - Expected result
- [ ] Test case 2 - Expected result
- [ ] Edge case - Expected result

### AI Prompt for This Phase

```
[Ready-to-use prompt for AI agent - see Prompting section]
```
```

### Phase Sizing Guidelines

| Size | Files | Context Tokens | Duration |
|------|-------|----------------|----------|
| Small | 1-2 | ~5k | 15-30 min |
| Medium | 3-5 | ~15k | 30-60 min |
| Large | 6-10 | ~30k | 1-2 hours |
| Too Large | 10+ | 50k+ | **Split it!** |

**Rule of thumb**: If a phase feels too big, it probably is. Split it.

---

## Self-Contained Phase Prompts

### The Zero-Context Rule

**Critical**: Each AI agent starts a phase with ZERO prior context. They haven't read previous phases. They don't know what files exist. They don't know your patterns.

**Every phase prompt must be completely self-contained.**

### What "Self-Contained" Means

The agent should be able to implement the phase by reading ONLY that phase's section. Include:

| Must Include | Why |
|--------------|-----|
| Project description | Agent doesn't know what you're building |
| Relevant tech stack | Agent needs to know frameworks/libraries |
| Database schema | If phase touches DB, show the tables |
| What exists from prerequisites | Describe files, components, APIs that exist |
| Code patterns to follow | Show snippets of existing patterns |
| Exact file paths | Don't say "create a component", say "create `src/components/Button.tsx`" |
| Success criteria | How agent knows it's done |
| Manual testing steps | How human verifies it works |

### Describing Prerequisites

Don't just list prerequisites—**describe what exists**:

❌ **Bad:**
```
Prerequisites: Phase 2 completed
```

✅ **Good:**
```
Prerequisites:
- Phase 2 completed
- Database has `flights` table (id, origin, destination, price, departure_time)
- Supabase clients exist at src/lib/supabase/{client,server}.ts
- Base UI components exist at src/components/ui/{Button,Input}.tsx
```

### Example: Full Self-Contained Prompt

```markdown
## Phase 5: Flight Search API

### Context
You are building FlightSearch, a web app for searching flights with flexible
date ranges and better filtering than existing tools (Kayak, Google Flights).

**Tech Stack:** Next.js 14 (App Router), Supabase, Tailwind CSS, TypeScript

### What Already Exists (Prerequisites)
From previous phases, these files/features exist:

**Database (Phase 2):**
- `user_preferences` table: id, user_id, default_origin, max_layover_minutes
- `saved_searches` table: id, user_id, origin, destination, date_from, date_to

**Auth (Phase 3):**
- `src/middleware.ts` - Auth middleware protecting routes
- `src/lib/hooks/useAuth.ts` - Hook returning { user, loading, signOut }

**UI Components (Phase 4):**
- `src/components/ui/Button.tsx` - Primary/secondary/ghost variants
- `src/components/ui/Input.tsx` - With label and error states

### Your Task
Create the flight search API that calls Amadeus and returns formatted results.

### Requirements

1. **Create Amadeus client**
   Create `src/lib/amadeus/client.ts`:
   - Initialize Amadeus SDK with env vars
   - Export configured client

2. **Create search API route**
   Create `src/app/api/flights/search/route.ts`:
   - Accept POST with { origin, destination, departureDate, returnDate, passengers }
   - Call Amadeus Flight Offers Search
   - Transform response to our format
   - Return JSON

3. **Create types**
   Create `src/types/flights.ts`:
   - FlightOffer interface
   - SearchParams interface
   - SearchResponse interface

### Files to Create
- `src/lib/amadeus/client.ts`
- `src/app/api/flights/search/route.ts`
- `src/types/flights.ts`

### Success Criteria
- [ ] API route accepts search parameters
- [ ] Amadeus is called correctly
- [ ] Response is transformed to clean format
- [ ] Errors are handled gracefully

### Manual Testing
- [ ] POST to /api/flights/search with valid params → returns flight offers
- [ ] POST with invalid params → returns 400 with error message
- [ ] Amadeus error → returns 500 with message (not raw error)
```

### Common Mistakes

❌ **Assuming context:**
```
"Use the auth hook we created earlier"
```
Agent doesn't know what hook or where it is.

✅ **Explicit reference:**
```
"Use the auth hook at src/lib/hooks/useAuth.ts which returns { user, loading, signOut }"
```

❌ **Vague file locations:**
```
"Create a new API route for search"
```

✅ **Exact paths:**
```
"Create src/app/api/flights/search/route.ts"
```

❌ **Missing schema:**
```
"Save the search to the database"
```

✅ **Include schema:**
```
"Save to saved_searches table (id UUID, user_id UUID, origin TEXT, destination TEXT, created_at TIMESTAMP)"
```

---

## Phase Design Principles

### 1. Single Responsibility
Each phase should do ONE thing well:
- ✅ "Set up authentication"
- ✅ "Create message list component"
- ❌ "Set up auth, create components, and add API routes"

### 2. Testable Outcome
Every phase must have a verifiable outcome:
- ✅ "Login page loads and accepts credentials"
- ✅ "API returns list of scores"
- ❌ "Database schema is correct" (how do you test this?)

### 3. Clear Boundaries
Define exactly what's in and out of scope:
```markdown
**In Scope:**
- Create the login form UI
- Connect to Supabase auth
- Redirect on success

**Out of Scope (handled in other phases):**
- Password reset flow
- Remember me functionality
- OAuth providers
```

### 4. Minimal Context Required
The AI should only need to know:
- What was built in prerequisite phases
- What this phase needs to accomplish
- Relevant code patterns from the project

**Don't include:**
- Full codebase history
- Unrelated feature details
- Future phase information

---

## Parallel Execution

### Identifying Parallel Opportunities

Phases can run in parallel when they:
1. Don't modify the same files
2. Don't depend on each other's output
3. Can be tested independently

### Parallel Execution Map

Create a visual dependency graph:

```
Phase 1 (Setup)
    │
    ▼
Phase 2 (Database)
    │
    ├─────────────┬─────────────┐
    ▼             ▼             ▼
Phase 3       Phase 4       Phase 5
(Auth)        (UI)          (API)
    │             │             │
    └──────┬──────┴──────┬──────┘
           │             │
           ▼             ▼
       Phase 6       Phase 7
       (Feature A)   (Feature B)
           │             │
           └──────┬──────┘
                  │
                  ▼
              Phase 8
              (Integration)
```

### Include Graph in Implementation Doc

**Always include the parallel execution map in your implementation document.** This lets anyone (human or AI) see at a glance:
- Which phases must be sequential
- Which phases can run in parallel
- Integration/merge points

Example format for your doc:

```markdown
## Parallel Execution Map

Phases 3, 4, 5 can run in parallel after Phase 2 completes.
Phases 8, 9, 10 can run in parallel after Phase 7 completes.

```
Phase 1 (Setup)
    │
    ▼
Phase 2 (Database)
    │
    ├───────────┬───────────┐
    ▼           ▼           ▼
Phase 3     Phase 4     Phase 5     ← Parallel
(Auth)      (UI)        (Data)
    │           │           │
    └─────┬─────┴───────────┘
          │
          ▼
      Phase 6 (Main View)            ← Requires 3, 4, 5
          │
    ├─────┼─────┬─────┐
    ▼     ▼     ▼     ▼
  Ph 7  Ph 8  Ph 9  Ph 10           ← Parallel
    │     │     │     │
    └─────┴─────┴─────┘
          │
          ▼
     Phase 11 (Polish)
```
```

### Running Parallel Phases

**Option A: Multiple terminal sessions**
- Open separate terminal windows
- Start different AI agents in each
- Run different phases simultaneously
- Test each independently, then proceed to merge phase

**Option B: Different AI providers**
- Phase 3 → Claude
- Phase 4 → Gemini
- Phase 5 → Claude
- Compare outputs for quality

**Option C: Sequential (simpler)**
- If phases are small enough, sequential might be faster
- Less merge conflict risk
- Easier to debug if something breaks

### Merge Points

After parallel phases complete, you often need an integration phase:
- Test that parallel work integrates correctly
- Resolve any conflicts or inconsistencies
- This is where bugs from parallel work surface

---

## Testing Between Phases

### Why Manual Testing Matters

- Catches issues before they compound
- Verifies AI understood the requirements
- Creates checkpoint for rollback
- Builds confidence for next phase

### Testing Checklist Template

```markdown
## Phase N Testing

### Setup
- [ ] Start dev server: `npm run dev`
- [ ] Clear browser cache/localStorage if needed
- [ ] Reset test database if needed

### Functional Tests
- [ ] [Action] → [Expected Result]
- [ ] [Action] → [Expected Result]
- [ ] [Edge case] → [Expected Result]

### Integration Tests
- [ ] Previous feature still works
- [ ] New feature integrates correctly

### Error Cases
- [ ] [Invalid input] → [Error handling works]
- [ ] [Network failure] → [Graceful degradation]

### Visual/UX
- [ ] UI matches expectations
- [ ] No console errors
- [ ] Responsive on mobile (if applicable)

### Sign-off
- [ ] All tests pass
- [ ] Ready for next phase
```

### What If Tests Fail?

1. **Don't proceed to next phase** - Fix issues first
2. **Document the fix** - Update phase instructions if needed
3. **Consider phase design** - Was the phase too complex?
4. **Rollback if needed** - Git is your friend

---

## Context Management

### Token Budget Awareness

Different AI models have different context limits:
- Claude 3.5 Sonnet: ~200k tokens
- Claude Opus: ~200k tokens
- GPT-4: ~128k tokens
- Gemini 1.5: ~1M tokens (but quality degrades)

**Rule of thumb**: Stay under 50% of context limit for best quality

### Strategies for Large Projects

1. **Summarize previous phases** - Don't include full code, summarize what exists

2. **Reference files, don't include** - Say "follow patterns in X" instead of pasting X

3. **Split complex phases** - Better two small phases than one massive one

4. **Use .md files as context** - Point AI to documentation instead of explaining everything

### Context Refresh Points

At certain milestones, start fresh:
- After major integration points
- When switching to unrelated feature area
- If AI seems confused or inconsistent
- After merging parallel work

---

## Example Workflow

### Project: Leaderboard Service

**Day 1: Planning**
1. Create main plan document
2. Create implementation phases document
3. Review and refine phase boundaries
4. Identify parallel opportunities

**Day 2: Foundation (Sequential)**
- Phase 1: Project setup ✓
- Phase 2: Database schema ✓
- Test: Tables exist, can query them

**Day 3: Core Features (Parallel)**
- Agent A: Phase 3 (Auth)
- Agent B: Phase 4 (API routes)
- Agent C: Phase 5 (Unity client)
- Test each independently

**Day 4: Integration**
- Phase 6: Connect auth to API
- Phase 7: Test Unity client with API
- Test: Full flow works

**Day 5: Polish**
- Phase 8: Error handling
- Phase 9: UI polish
- Final testing

---

## Quick Reference

### Workflow
```
1. Create CLAUDE.md + GEMINI.md in project root (same content, points to plans)
2. Create plans/1_project.md (main plan)
3. Create plans/1_project-implementation.md (phases)
4. Start new agent (Claude or Gemini) → "implement phase 3"
5. Agent reads its .md file → finds plans → executes phase
6. You test → update status in both .md files → next phase
```

### Phase Sizing
- 1-2 files = Small (15-30 min)
- 3-5 files = Medium (30-60 min)
- 6+ files = Consider splitting

### Self-Contained Prompts Must Include
- Project description
- Tech stack
- What exists from prerequisites (files, tables, APIs)
- Exact file paths to create/modify
- Success criteria
- Manual testing steps

### Prompt Structure
1. Context (project + what exists)
2. Task (specific goal)
3. Requirements (detailed steps)
4. Acceptance criteria (testable)

### Testing Cadence
- After EVERY phase
- No exceptions
- Don't batch testing

### Parallel Safety
- Different files = Safe
- Same file = Sequential
- Integration = After parallel completes

---

## Templates

### CLAUDE.md / GEMINI.md Template

Create both files with identical content:

```markdown
# [Project Name]

> [One-line description]

## Phased Development

- **Main Plan:** `plans/1_[name].md`
- **Implementation:** `plans/1_[name]-implementation.md`

When asked to "implement phase N", read the implementation doc and execute that phase.

### Status
- [x] Phase 1: Setup
- [ ] Phase 2: Database
- [ ] Phase 3: Auth
...

## Tech Stack
- [List technologies]

## Notes
- [Project-specific conventions]
```

**Tip:** Use a symlink to keep them in sync:
```bash
ln -s CLAUDE.md GEMINI.md
```

### Quick Phase Template
```markdown
## Phase N: [Name]

**Goal:** [One sentence]

**Prerequisites:**
- Phase X completed
- [Describe what exists: files, tables, APIs]

**Steps:**
1. Create `path/to/file.ts` - [what it does]
2. ...

**Files Created:**
- `path/to/file.ts`

**Success Criteria:**
- [ ] [Criterion 1]
- [ ] [Criterion 2]

**Manual Testing:**
- [ ] [Action] → [Expected result]
- [ ] [Action] → [Expected result]
```

### Minimal Prompt (when you just need to say it)
```
implement phase 3
```

Agent reads its instruction file (CLAUDE.md or GEMINI.md), finds plans, executes phase.

---

*This guide is a living document. Update it as you discover what works best for your workflow.*
