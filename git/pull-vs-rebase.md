# Git Pull vs Git Pull --rebase

## The Problem

When your local branch and the remote branch have both moved forward independently, they've **diverged**. Git needs to know how to combine them.

```
Remote main:  A --- B --- C --- D  (2 new commits from merged PRs)

Local main:   A --- B --- X --- Y  (2 commits you made locally)
```

Both branches share commits A and B, but then went in different directions. Git has three strategies to reconcile this.

---

## Strategy 1: Merge (`git pull --no-rebase`)

This creates a **merge commit**—a special commit that has two parents and ties both histories together.

```
               C --- D ----
              /            \
A --- B -----+              +--- M  (merge commit)
              \            /
               X --- Y ----
```

**The merge commit (M) doesn't combine or squash your commits.** All commits stay separate and intact (C, D, X, Y all remain as individual commits). The merge commit M is just a "meeting point" that says "these two lines of history are now joined."

Your final history will have: A, B, C, D, X, Y, M (7 commits total). The merge commit M contains no actual code changes itself (assuming no conflicts)—it's purely structural.

**When to use**: When you want to preserve the exact historical record of what happened and when.

---

## Strategy 2: Rebase (`git pull --rebase`)

This temporarily removes your local commits, pulls the remote changes, then replays your commits on top.

```
Step 1: Git sets aside your commits X and Y
Step 2: Git pulls remote (now you have A --- B --- C --- D)
Step 3: Git replays X on top of D (becomes X')
Step 4: Git replays Y on top of X' (becomes Y')

Final result:
A --- B --- C --- D --- X' --- Y'
```

Think of it like a stash workflow:
1. Stash (uncommit) your changes
2. Pull to get up to date with remote
3. Pop your changes back and recommit them

The commits get new hashes (X becomes X', Y becomes Y') because they now have different parent commits, but the actual code changes are identical.

**When to use**: When you want a clean, linear history. This is the most common choice for everyday work.

---

## Strategy 3: Fast-Forward Only (`git pull --ff-only`)

This only works when there's **nothing to reconcile**—your local branch has no commits that remote doesn't have.

```
This works (you have no local commits):
Local:  A --- B
Remote: A --- B --- C --- D

Git just slides your pointer forward:
Local:  A --- B --- C --- D   ✓
```

```
This FAILS (you have local commits):
Local:  A --- B --- X --- Y
Remote: A --- B --- C --- D

Git refuses: "I can't fast-forward, you have local commits!"   ✗
```

**Important**: This doesn't prevent you from committing locally. You can commit all you want. It only affects what happens when you try to `git pull`. If your local and remote have diverged, ff-only refuses to pull and forces you to manually decide (rebase or merge) each time.

**When to use**: When you want to be explicit about every merge/rebase decision rather than having Git pick automatically.

---

## Setting a Default

To avoid seeing the "divergent branches" error, set a global preference:

```bash
# For clean linear history (recommended for most people)
git config --global pull.rebase true

# For preserving merge history
git config --global pull.rebase false

# To be forced to decide each time
git config --global pull.ff only
```

Or override per-pull:
```bash
git pull --rebase
git pull --no-rebase
git pull --ff-only
```

---

## Quick Reference

| Strategy | Command | Result | Use When |
|----------|---------|--------|----------|
| Merge | `git pull --no-rebase` | Creates merge commit, keeps all commits separate | You want full historical accuracy |
| Rebase | `git pull --rebase` | Linear history, your commits moved to the end | You want clean history (most common) |
| FF-only | `git pull --ff-only` | Fails if diverged, forces manual decision | You want explicit control |

For most day-to-day work, `git pull --rebase` is the cleanest option.
