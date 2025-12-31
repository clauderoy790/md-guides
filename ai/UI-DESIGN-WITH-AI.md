# Communicating UI Design to AI

> A guide for finding design inspiration and effectively communicating it to AI assistants (Claude, Gemini, ChatGPT) for any project with a user interface.

---

## Table of Contents

1. [The Challenge](#the-challenge)
2. [Finding Design Inspiration](#finding-design-inspiration)
3. [Communication Methods](#communication-methods)
4. [Best Practices](#best-practices)
5. [Prompt Templates](#prompt-templates)
6. [Common Mistakes](#common-mistakes)

---

## The Challenge

AI can write excellent code but can't read your mind about aesthetics. Saying "make it look good" gives inconsistent results. You need to communicate:

- **Visual style** - Colors, spacing, typography, shapes
- **Functional layout** - Where things go, how they're organized
- **Mood/vibe** - Professional, playful, minimal, bold
- **What to avoid** - Equally important as what you want

---

## Finding Design Inspiration

### Design Galleries

| Site | Best For | How to Use |
|------|----------|------------|
| [Dribbble](https://dribbble.com) | Visual inspiration, UI concepts | Search "dashboard", "admin panel", "SaaS" |
| [Mobbin](https://mobbin.com) | Real app screenshots | Browse by app type, filter by platform |
| [Behance](https://behance.net) | Full case studies | Search project type + "UI" |
| [Awwwards](https://awwwards.com) | Cutting-edge web design | Browse winners, filter by style |
| [Landingfolio](https://landingfolio.com) | Landing pages | Filter by industry/style |
| [SaaS Landing Page](https://saaslandingpage.com) | SaaS-specific designs | Great for product dashboards |
| [Refero](https://refero.design) | Real product screenshots | Organized by product type |

### Reference Real Products

Sometimes the best reference is a product you already use:

**Dashboards/Admin:**
- [Linear](https://linear.app) - Minimal, keyboard-first, dark
- [Vercel](https://vercel.com/dashboard) - Clean, modern, good dark mode
- [Supabase](https://supabase.com/dashboard) - Developer-friendly, functional
- [Stripe](https://dashboard.stripe.com) - Professional, dense but clear
- [Notion](https://notion.so) - Flexible, content-focused
- [GitHub](https://github.com) - Utilitarian, familiar to devs
- [Raycast](https://raycast.com) - Polished, macOS-native feel
- [Planetscale](https://planetscale.com) - Modern, good typography

**Marketing/Landing:**
- [Stripe](https://stripe.com) - Gradients, animations, premium
- [Linear](https://linear.app) - Minimal, bold typography
- [Vercel](https://vercel.com) - Dark, technical, modern
- [Tailwind](https://tailwindcss.com) - Clean, documentation-focused

### Quick Inspiration Workflow

1. Go to Dribbble or Mobbin
2. Search: `[your project type] minimal` (e.g., "dashboard minimal")
3. Scroll until something catches your eye
4. Screenshot 2-3 designs you like
5. Note what specifically you like about each

---

## Communication Methods

### Method 1: Screenshots (Most Effective)

**How:** Take screenshots of designs you like and share directly with AI.

**Why it works:** AI (Claude, GPT-4) can analyze images and extract:
- Color palettes
- Layout patterns
- Component styles
- Spacing relationships
- Typography choices

**Example prompt:**
```
Look at this screenshot. I like:
- The card style with subtle borders
- The sidebar navigation layout
- The muted color palette

Create a similar style for my dashboard. Don't copy exactly,
but use these as inspiration for the overall feel.
```

**Pro tip:** Annotate screenshots to highlight specific elements you like.

---

### Method 2: Reference by Name

**How:** Name specific products or sites.

**Example:**
```
Style the dashboard similar to Linear's UI:
- Minimal, lots of whitespace
- Subtle borders instead of shadows
- Keyboard shortcut hints
- Command palette style
```

**Works best when:**
- The AI knows the product (popular tools)
- You describe what specifically you like
- You mention what to skip from that design

---

### Method 3: Style Specification

**How:** Define explicit design tokens.

**Example:**
```markdown
## Style Specification

**Colors:**
- Background: #fafafa (light), #0a0a0a (dark)
- Cards: white with 1px #e5e5e5 border
- Primary: #2563eb (blue-600)
- Text: #171717 (primary), #737373 (secondary)

**Typography:**
- Font: Inter or system-ui
- Headings: 600 weight
- Body: 400 weight, 16px base

**Spacing:**
- Page padding: 24px
- Card padding: 16px
- Gap between elements: 12px

**Borders:**
- Radius: 8px (cards), 6px (buttons), 4px (inputs)
- Color: #e5e5e5

**Effects:**
- No shadows (use borders instead)
- No gradients
- Subtle hover states (background color change)
```

**Works best for:**
- Maintaining consistency across sessions
- Teams with multiple people prompting AI
- Projects needing strict brand adherence

---

### Method 4: Vibe/Mood Description

**How:** Use adjectives and comparisons.

**Vocabulary:**

| Category | Options |
|----------|---------|
| **Density** | Spacious, comfortable, compact, dense |
| **Mood** | Professional, playful, serious, friendly, technical |
| **Style** | Minimal, bold, subtle, clean, busy, ornate |
| **Era** | Modern, classic, retro, futuristic |
| **Feel** | Corporate, startup, indie, enterprise, consumer |

**Example:**
```
Design a dashboard that feels:
- Professional but not corporate
- Minimal but not empty
- Technical but approachable
- Like a well-made developer tool, not a marketing site
```

---

### Method 5: Anti-Patterns (What to Avoid)

Sometimes it's easier to say what you don't want:

```
Avoid:
- Gradients (especially blue-purple AI gradients)
- Drop shadows on everything
- Rounded corners larger than 12px
- Bright/saturated colors
- Animations that don't serve a purpose
- "Glassmorphism" or frosted glass effects
- Overly playful illustrations
- Stock photos of people
```

---

## Best Practices

### 1. Create a Style Guide First

Before building any UI, create a `UI-STYLE-GUIDE.md`:

```markdown
# [Project Name] UI Style Guide

## References
- Primary inspiration: [Screenshot or link]
- Secondary inspiration: [Screenshot or link]

## Colors
[Define your palette]

## Typography
[Define fonts and sizes]

## Components
[Describe key component styles]

## Do's and Don'ts
[List specific preferences]
```

Then reference it in every prompt:
```
Follow the style guide at docs/UI-STYLE-GUIDE.md
```

### 2. Be Specific About What You Like

Bad: "I like this design"
Good: "I like the card borders, the spacing between sections, and the muted blue accent color"

### 3. Provide Context

Tell the AI:
- What the project is
- Who the users are
- What feeling you want users to have

### 4. Iterate Visually

1. Ask AI to generate initial design
2. Screenshot the result
3. Annotate what to change
4. Share annotated screenshot
5. Repeat

### 5. Use Consistent Terminology

Pick terms and stick with them:
- "Cards" not sometimes "boxes" and sometimes "containers"
- "Primary button" not sometimes "main button" or "CTA"
- "Sidebar" not sometimes "nav" or "menu"

---

## Prompt Templates

### Template 1: Starting a New UI Project

```markdown
# Task: Create UI Style Guide for [Project Name]

## Project Context
[1-2 sentences about what the project is]

## Target Users
[Who will use this UI]

## My Preferences
- Mode: [Light/Dark/Both]
- Vibe: [e.g., Minimal, professional, friendly]
- References: [Products you like, e.g., "Similar to Linear"]

## Must Avoid
- [List things you don't want]

## Attachments
[Screenshots of designs you like - describe what you like about each]

## Deliverable
Create a UI style guide including:
1. Color palette (with specific hex values)
2. Typography scale
3. Spacing system
4. Component styles (buttons, cards, inputs, etc.)
5. Tailwind config customizations (if using Tailwind)
```

### Template 2: Building a Specific Page

```markdown
# Task: Design [Page Name]

## Context
- Project: [Name]
- Style guide: [Path to style guide]
- This page is for: [Purpose]

## Requirements
- [List functional requirements]
- [What data/content it shows]
- [Key actions users can take]

## Layout Preferences
[Describe layout or attach wireframe]

## Reference
[Screenshot of similar page you like, or description]

Follow the style guide exactly. Don't introduce new colors or patterns.
```

### Template 3: Refining Existing UI

```markdown
# Task: Improve This UI

## Current State
[Screenshot of current UI]

## What's Wrong
- [Specific issues]

## What I Want
- [Specific improvements]

## Reference
[Screenshot of desired style, if helpful]

Keep the layout the same, just improve the visual styling.
```

### Template 4: Quick Style Matching

```markdown
Look at this screenshot: [attach image]

Extract:
1. Color palette (hex values)
2. Border radius values
3. Shadow styles
4. Typography (font, sizes, weights)
5. Spacing patterns

Format as Tailwind config and CSS variables.
```

---

## Common Mistakes

### 1. Being Too Vague
❌ "Make it look modern"
✅ "Use a minimal style with subtle borders, no shadows, and a blue-600 accent color"

### 2. Not Providing References
❌ "Design a nice dashboard"
✅ "Design a dashboard similar to Linear's style - here's a screenshot of what I like about it"

### 3. Contradicting Yourself
❌ "Make it minimal but also feature-rich with lots of information"
✅ "Show dense information but use minimal visual styling - fewer colors, subtle borders, good typography hierarchy"

### 4. Forgetting Mobile
❌ Only describing desktop layout
✅ "Desktop: sidebar navigation. Mobile: bottom tab bar or hamburger menu"

### 5. Not Iterating
❌ Expecting perfection on first try
✅ Treating it as a conversation - refine based on output

### 6. Overloading the Prompt
❌ 2000 words describing every detail upfront
✅ Start simple, add specifics as needed

---

## Quick Reference

### Minimum Viable Design Prompt

```
Create [component/page] for [project].
Style: [1-2 adjectives]
Reference: [product name or screenshot]
Avoid: [1-2 things]
```

### Example

```
Create a settings page for a leaderboard admin dashboard.
Style: Minimal, professional
Reference: Similar to Vercel's dashboard settings
Avoid: Shadows, bright colors
```

---

## Tools That Help

| Tool | Purpose |
|------|---------|
| **CleanShot X / ShareX** | Better screenshots with annotations |
| **Figma** | Quick wireframes to share |
| **Excalidraw** | Hand-drawn style wireframes |
| **Coolors.co** | Generate color palettes |
| **Realtime Colors** | Preview color schemes on a fake UI |
| **Font Pair** | Find font combinations |

---

*The best design communication is visual. When in doubt, screenshot it.*
