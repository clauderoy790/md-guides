# GitHub CLI Guide

> Master the `gh` command to manage GitHub entirely from your terminal.

---

## Table of Contents

1. [Installation](#installation)
2. [Authentication](#authentication)
3. [Repository Management](#repository-management)
4. [Collaborators](#collaborators)
5. [Pull Requests](#pull-requests)
6. [Issues](#issues)
7. [GitHub Actions & CI/CD](#github-actions--cicd)
8. [Gists](#gists)
9. [Releases](#releases)
10. [SSH Keys](#ssh-keys)
11. [Aliases](#aliases)
12. [API Access](#api-access)
13. [Quick Reference](#quick-reference)

---

## Installation

### macOS (Homebrew)

```bash
brew install gh
```

### Verify Installation

```bash
gh --version
```

### Update

```bash
brew upgrade gh
```

---

## Authentication

Before using `gh`, you need to authenticate with GitHub.

### Interactive Login (Recommended)

```bash
gh auth login
```

You'll be prompted to:
1. Choose GitHub.com or GitHub Enterprise
2. Choose HTTPS or SSH protocol
3. Authenticate via browser or paste a token

### Check Auth Status

```bash
gh auth status
```

### Login with Token

```bash
gh auth login --with-token < token.txt
```

Or paste directly:

```bash
echo "ghp_xxxxxxxxxxxx" | gh auth login --with-token
```

### Switch Accounts

```bash
gh auth logout
gh auth login
```

### Refresh Token (if expired)

```bash
gh auth refresh
```

---

## Repository Management

### Create a New Repository

**This is the big one** - skip the browser entirely.

```bash
# Interactive mode - prompts for everything
gh repo create

# Create and clone in one step
gh repo create my-project --clone

# Public repo
gh repo create my-project --public --clone

# Private repo
gh repo create my-project --private --clone

# Create from current directory (existing local repo)
gh repo create --source=. --public --push
```

#### Create Options

| Flag | Description |
|------|-------------|
| `--public` | Make repo public |
| `--private` | Make repo private |
| `--clone` | Clone after creating |
| `--source=.` | Use current directory as source |
| `--push` | Push local commits after creating |
| `--description "text"` | Set description |
| `--gitignore node` | Add .gitignore template |
| `--license mit` | Add license |

#### Common Workflows

```bash
# Start fresh project
gh repo create awesome-project --public --clone --gitignore node --license mit
cd awesome-project

# Push existing local project to new GitHub repo
cd my-local-project
gh repo create --source=. --public --push

# Create from template
gh repo create my-project --template owner/template-repo --clone
```

### Clone a Repository

```bash
# Simple - no need for full URL
gh repo clone owner/repo

# Clone your own repo (just the name)
gh repo clone my-repo

# Clone to specific directory
gh repo clone owner/repo ./my-directory

# Clone and set upstream (for forks)
gh repo clone owner/repo -- --origin upstream
```

### View Repository Info

```bash
# View current repo info
gh repo view

# View in browser
gh repo view --web

# View specific repo
gh repo view owner/repo
```

### Open in Browser

```bash
# Open current repo in browser
gh browse

# Open specific file
gh browse src/index.js

# Open at specific line
gh browse src/index.js:42

# Open issues page
gh browse --issues

# Open PRs page
gh browse --pulls

# Open settings
gh browse --settings
```

### Fork a Repository

```bash
# Fork and clone
gh repo fork owner/repo --clone

# Fork only (no clone)
gh repo fork owner/repo

# Fork to specific org
gh repo fork owner/repo --org my-org
```

### Repository Settings

```bash
# Edit description
gh repo edit --description "New description"

# Change visibility
gh repo edit --visibility public
gh repo edit --visibility private

# Add topics
gh repo edit --add-topic cli --add-topic golang

# Enable/disable features
gh repo edit --enable-wiki=false
gh repo edit --enable-issues=true

# Set default branch
gh repo edit --default-branch main
```

### List Repositories

```bash
# Your repos
gh repo list

# Specific user's repos
gh repo list username

# Organization repos
gh repo list my-org

# With filters
gh repo list --language go --limit 20
gh repo list --source  # non-forks only
```

### Delete a Repository

```bash
gh repo delete owner/repo --yes
```

---

## Collaborators

There's no dedicated `gh` subcommand for collaborators, but you can manage them via the API.

### Add a Collaborator

```bash
# Add collaborator (sends invitation by GitHub username)
gh api -X PUT repos/owner/repo/collaborators/USERNAME

# With specific permission level
gh api -X PUT repos/owner/repo/collaborators/USERNAME -f permission=push
```

#### Permission Levels

| Permission | Access |
|------------|--------|
| `pull` | Read-only access |
| `push` | Read and write access (default) |
| `triage` | Manage issues and PRs without write access |
| `maintain` | Maintainer access (manage repo without admin) |
| `admin` | Full admin access |

> **GitHub Free Limitation:** For private repos on GitHub Free, collaborators only get read/write access (`push`). Granular permissions like `pull` (read-only) on private repos require GitHub Team or Enterprise. Public repos support all permission levels.

### List Collaborators

```bash
# List all collaborators
gh api repos/owner/repo/collaborators

# With their permission level
gh api repos/owner/repo/collaborators --jq '.[] | {login: .login, permission: .role_name}'
```

### Check Specific User's Permission

```bash
gh api repos/owner/repo/collaborators/USERNAME/permission
```

### Remove a Collaborator

```bash
gh api -X DELETE repos/owner/repo/collaborators/USERNAME
```

### Notes

- Collaborators are added by **GitHub username**, not email
- Adding a collaborator sends them an invitation they must accept
- To invite by email (for users without GitHub accounts), use the web interface

---

## Pull Requests

### Create a Pull Request

```bash
# Interactive mode
gh pr create

# With title and body
gh pr create --title "Add feature X" --body "Description here"

# Draft PR
gh pr create --draft

# Auto-fill from commits
gh pr create --fill

# Target specific branch
gh pr create --base develop

# Assign reviewers
gh pr create --reviewer user1,user2

# Add labels
gh pr create --label bug,urgent

# Link to issue (auto-closes when merged)
gh pr create --title "Fix login bug" --body "Fixes #123"
```

### List Pull Requests

```bash
# All open PRs
gh pr list

# Your PRs
gh pr list --author @me

# PRs needing your review
gh pr list --search "review-requested:@me"

# All states
gh pr list --state all

# Merged PRs
gh pr list --state merged

# With labels
gh pr list --label bug
```

### View a Pull Request

```bash
# View current branch's PR
gh pr view

# View specific PR
gh pr view 123

# Open in browser
gh pr view 123 --web

# View diff
gh pr diff 123
```

### Checkout a Pull Request

```bash
# Switch to PR branch
gh pr checkout 123

# Checkout and create local branch
gh pr checkout 123 --branch my-local-branch
```

### Review a Pull Request

```bash
# Start review
gh pr review 123

# Approve
gh pr review 123 --approve

# Request changes
gh pr review 123 --request-changes --body "Please fix X"

# Comment
gh pr review 123 --comment --body "Looks good but..."
```

### Merge a Pull Request

```bash
# Interactive (prompts for method)
gh pr merge 123

# Merge commit
gh pr merge 123 --merge

# Squash and merge
gh pr merge 123 --squash

# Rebase and merge
gh pr merge 123 --rebase

# Auto-merge when checks pass
gh pr merge 123 --auto --squash

# Delete branch after merge
gh pr merge 123 --squash --delete-branch
```

### Close/Reopen PR

```bash
gh pr close 123
gh pr reopen 123
```

### PR Checks Status

```bash
# View CI/checks status
gh pr checks 123

# Wait for checks to complete
gh pr checks 123 --watch
```

---

## Issues

### Create an Issue

```bash
# Interactive
gh issue create

# With title and body
gh issue create --title "Bug: X not working" --body "Steps to reproduce..."

# With labels and assignee
gh issue create --title "Bug" --label bug,urgent --assignee @me

# From file
gh issue create --title "Feature request" --body-file description.md
```

### List Issues

```bash
# All open issues
gh issue list

# Assigned to you
gh issue list --assignee @me

# By label
gh issue list --label bug

# All states
gh issue list --state all

# Search
gh issue list --search "auth error"
```

### View an Issue

```bash
gh issue view 42

# Open in browser
gh issue view 42 --web
```

### Close/Reopen Issues

```bash
# Close
gh issue close 42

# Close with comment
gh issue close 42 --comment "Fixed in #123"

# Reopen
gh issue reopen 42
```

### Edit Issues

```bash
# Add labels
gh issue edit 42 --add-label bug

# Remove labels
gh issue edit 42 --remove-label wontfix

# Assign
gh issue edit 42 --add-assignee username

# Edit title
gh issue edit 42 --title "New title"
```

### Comment on Issues

```bash
gh issue comment 42 --body "I can reproduce this"
```

### Link Issues to PRs

When you create a PR, use keywords in the body to auto-close issues:

```bash
gh pr create --title "Fix auth bug" --body "Fixes #42"
```

Keywords: `Fixes`, `Closes`, `Resolves` (case-insensitive)

---

## GitHub Actions & CI/CD

### List Workflow Runs

```bash
# Recent runs
gh run list

# Specific workflow
gh run list --workflow build.yml

# Filter by status
gh run list --status failure

# Filter by branch
gh run list --branch main
```

### View a Workflow Run

```bash
# View run details
gh run view 12345

# View with logs
gh run view 12345 --log

# View failed logs only
gh run view 12345 --log-failed

# Open in browser
gh run view 12345 --web
```

### Watch a Run in Real-Time

```bash
# Watch current branch's run
gh run watch

# Watch specific run
gh run watch 12345
```

### Re-run Failed Jobs

```bash
# Re-run all jobs
gh run rerun 12345

# Re-run only failed jobs
gh run rerun 12345 --failed
```

### Cancel a Run

```bash
gh run cancel 12345
```

### Trigger a Workflow

```bash
# Trigger workflow_dispatch event
gh workflow run build.yml

# With inputs
gh workflow run deploy.yml -f environment=staging

# On specific branch
gh workflow run build.yml --ref feature-branch
```

### List Workflows

```bash
gh workflow list
```

### View Workflow

```bash
gh workflow view build.yml
```

---

## Gists

### Create a Gist

```bash
# From file
gh gist create file.txt

# Multiple files
gh gist create file1.txt file2.txt

# From stdin
echo "Hello World" | gh gist create

# Public gist
gh gist create file.txt --public

# With description
gh gist create file.txt --desc "My useful snippet"

# With custom filename
cat script.sh | gh gist create --filename "deploy.sh"
```

### List Your Gists

```bash
gh gist list

# Public only
gh gist list --public

# Secret only
gh gist list --secret
```

### View a Gist

```bash
gh gist view abc123

# Open in browser
gh gist view abc123 --web
```

### Edit a Gist

```bash
# Opens in editor
gh gist edit abc123

# Add file
gh gist edit abc123 --add newfile.txt
```

### Delete a Gist

```bash
gh gist delete abc123
```

### Clone a Gist

```bash
gh gist clone abc123
```

---

## Releases

### Create a Release

```bash
# Create from tag
gh release create v1.0.0

# With title and notes
gh release create v1.0.0 --title "Version 1.0" --notes "First stable release"

# From notes file
gh release create v1.0.0 --notes-file CHANGELOG.md

# Auto-generate notes from commits
gh release create v1.0.0 --generate-notes

# Draft release
gh release create v1.0.0 --draft

# Pre-release
gh release create v1.0.0-beta --prerelease

# With assets
gh release create v1.0.0 ./dist/*.zip
```

### List Releases

```bash
gh release list
```

### View a Release

```bash
gh release view v1.0.0

# Open in browser
gh release view v1.0.0 --web
```

### Download Release Assets

```bash
# Download all assets
gh release download v1.0.0

# Specific asset
gh release download v1.0.0 --pattern "*.zip"

# To specific directory
gh release download v1.0.0 --dir ./downloads
```

### Delete a Release

```bash
gh release delete v1.0.0 --yes
```

---

## SSH Keys

### Add SSH Key to GitHub

```bash
# Add existing key
gh ssh-key add ~/.ssh/id_ed25519.pub

# With title
gh ssh-key add ~/.ssh/id_ed25519.pub --title "MacBook Pro"
```

### List SSH Keys

```bash
gh ssh-key list
```

### Delete SSH Key

```bash
gh ssh-key delete 12345
```

### Generate and Add New Key

```bash
# Generate new key
ssh-keygen -t ed25519 -C "your_email@example.com"

# Add to GitHub
gh ssh-key add ~/.ssh/id_ed25519.pub --title "New Laptop"
```

---

## Aliases

Create shortcuts for common commands.

### Create Alias

```bash
# Simple alias
gh alias set co 'pr checkout'

# Now use: gh co 123

# Alias with arguments
gh alias set pv 'pr view --web'

# Complex alias
gh alias set bugs 'issue list --label bug --state open'
```

### List Aliases

```bash
gh alias list
```

### Delete Alias

```bash
gh alias delete co
```

### Useful Alias Ideas

```bash
# Quick PR checkout
gh alias set co 'pr checkout'

# My open PRs
gh alias set mypr 'pr list --author @me'

# PRs to review
gh alias set review 'pr list --search "review-requested:@me"'

# Today's activity
gh alias set today 'pr list --search "updated:>=$(date -v-1d +%Y-%m-%d)"'

# Clone my repos easily
gh alias set clone-mine 'repo clone'

# Quick issue creation
gh alias set bug 'issue create --label bug'

# View repo in browser
gh alias set open 'browse'
```

---

## API Access

For anything not covered by built-in commands.

### Basic API Calls

```bash
# GET request
gh api repos/owner/repo

# Get specific fields with jq
gh api repos/owner/repo --jq '.stargazers_count'

# POST request
gh api repos/owner/repo/issues -f title="New issue" -f body="Description"
```

### Useful API Examples

```bash
# Get your user info
gh api user

# List repo collaborators
gh api repos/owner/repo/collaborators

# Get PR comments
gh api repos/owner/repo/pulls/123/comments

# Star a repo
gh api -X PUT user/starred/owner/repo

# Get rate limit status
gh api rate_limit
```

### Pagination

```bash
# Auto-paginate
gh api repos/owner/repo/issues --paginate

# Limit results
gh api repos/owner/repo/issues --paginate -q '.[:10]'
```

### GraphQL

```bash
gh api graphql -f query='
  query {
    viewer {
      login
      repositories(first: 10) {
        nodes {
          name
          stargazerCount
        }
      }
    }
  }
'
```

---

## Quick Reference

### Installation & Auth

```bash
brew install gh              # Install
gh auth login                 # Login
gh auth status               # Check auth
```

### Repo Management

```bash
gh repo create name --public --clone    # Create + clone
gh repo create --source=. --push        # Push existing to new
gh repo clone owner/repo                # Clone
gh browse                               # Open in browser
gh repo view                            # View info
```

### Pull Requests

```bash
gh pr create --fill           # Create PR (auto-fill from commits)
gh pr list                    # List PRs
gh pr checkout 123            # Switch to PR branch
gh pr diff 123                # View diff
gh pr merge 123 --squash      # Squash merge
gh pr checks 123 --watch      # Watch CI status
```

### Issues

```bash
gh issue create               # Create issue
gh issue list --assignee @me  # My issues
gh issue close 42             # Close issue
```

### CI/CD

```bash
gh run list                   # List runs
gh run view 123 --log         # View logs
gh run watch                  # Watch current run
gh run rerun 123 --failed     # Re-run failed
```

### Other

```bash
gh gist create file.txt       # Create gist
gh release create v1.0.0      # Create release
gh ssh-key add key.pub        # Add SSH key
gh alias set co 'pr checkout' # Create alias
gh api user                   # API call
```

### Keyboard Shortcuts for Interactive Mode

When using interactive prompts:
- `Enter` - Select/confirm
- `Tab` - Toggle selection (multi-select)
- `↑/↓` - Navigate options
- `Ctrl+C` - Cancel

---

## Environment Variables

| Variable | Description |
|----------|-------------|
| `GH_TOKEN` | Auth token (alternative to `gh auth`) |
| `GH_HOST` | GitHub Enterprise hostname |
| `GH_REPO` | Override current repo context |
| `GH_EDITOR` | Editor for composing text |
| `GH_PAGER` | Pager for output (default: system pager) |
| `NO_COLOR` | Disable color output |

---

## Configuration

```bash
# View config
gh config list

# Set editor
gh config set editor vim

# Set default git protocol
gh config set git_protocol ssh

# Set default browser
gh config set browser firefox

# Disable prompts (for scripts)
gh config set prompt disabled
```

---

*Stop context-switching to the browser. `gh` brings GitHub to your terminal.*
