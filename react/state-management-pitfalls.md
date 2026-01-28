# React State Management Pitfalls

Understanding `useState`, `useCallback`, dependency arrays, and common bugs.

## Key Concepts

### 1. Side Effects

A **general programming term** (not React-specific).

**Pure function** - only computes a result from its inputs:
```javascript
// Pure - same input always gives same output, does nothing else
function add(a, b) {
  return a + b;
}
```

**Side effect** - when a function does something *beyond* returning a value:
```javascript
// Has side effects - modifies external state
function addAndLog(a, b) {
  console.log("Adding!");      // Side effect: I/O
  totalCount++;                // Side effect: modifies external variable
  saveToDatabase(a + b);       // Side effect: network call
  return a + b;
}
```

Common side effects: logging, API calls, modifying variables outside the function, updating DOM, setting timers.

---

### 2. The `prev` Pattern in useState

When your new state depends on the previous state, use the callback form:

```javascript
// ❌ Without prev - can have race conditions
setCount(count + 1);
setCount(count + 1);  // Both see same `count`, only adds 1 total!

// ✅ With prev - always correct
setCount(prev => prev + 1);
setCount(prev => prev + 1);  // Each sees latest value, adds 2 total
```

**Important:** The callback passed to setState should be **pure** - it should only compute and return the new value, with no side effects.

---

### 3. useCallback and Dependency Arrays

#### The Problem: Functions Are Recreated Every Render

```javascript
function Dashboard() {
  const [achievements, setAchievements] = useState([...]);

  // This function is recreated on EVERY render
  const handleDelete = (id) => {
    console.log(achievements);  // Uses achievements from this render
  };
}
```

#### The Solution: useCallback

`useCallback` memoizes the function, only recreating it when dependencies change:

```javascript
// Only recreate when `achievements` changes
const handleDelete = useCallback((id) => {
  console.log(achievements);
}, [achievements]);  // ← Dependency array
```

#### What Different Dependency Arrays Mean

```javascript
// ❌ Empty array: function created ONCE, never updates
useCallback((id) => {
  console.log(achievements);  // Will always see the INITIAL achievements!
}, []);

// ✅ With dependencies: function recreated when achievements changes
useCallback((id) => {
  console.log(achievements);  // Will see CURRENT achievements
}, [achievements]);

// 🤷 No useCallback: function recreated on EVERY render (works, but wasteful)
const handleDelete = (id) => {
  console.log(achievements);
};
```

#### Visual Example

```
Render 1: achievements = [A, B, C]
├─ handleDelete created, sees [A, B, C] ✓

Render 2: achievements = [A, B, C, D]  (added one)
├─ With [achievements]: NEW handleDelete created, sees [A, B, C, D] ✓
├─ With []:             SAME handleDelete, still sees [A, B, C] ✗ STALE!

Render 3: achievements = [A, B, D]  (deleted C)
├─ With [achievements]: NEW handleDelete created, sees [A, B, D] ✓
├─ With []:             SAME handleDelete, still sees [A, B, C] ✗ STALE!
```

---

## Real-World Bug: Double State Update

### The Problem

When deleting an achievement worth 3 XP:
- Expected: 33 → 30 XP
- Actual: 33 → 27 XP (double subtraction!)
- After page refresh: 30 XP (correct)

### The Buggy Code

```javascript
const handleAchievementDeleted = useCallback((id) => {
  setAchievements((prev) => {
    const deleted = prev.find((a) => a.id === id);
    setCurrentTotalXp((xp) => xp - deleted.points);  // ← Side effect inside!
    return prev.filter((a) => a.id !== id);
  });
}, []);  // ← Empty dependencies
```

### Why It Broke

The callback passed to `setAchievements` should be **pure** - it should only compute and return the new achievements array. But we put a side effect inside (`setCurrentTotalXp`).

Combined with React's state batching and Next.js's `revalidatePath()` (which refetches page data), this caused unpredictable behavior where XP was subtracted twice.

### Visual Explanation

```
BEFORE (buggy):
┌─────────────────────────────────┐
│ setAchievements((prev) => {     │
│   setCurrentTotalXp(...)  ← 💥  │  Side effect inside setState
│   return filtered array         │  = unpredictable behavior
│ })                              │
└─────────────────────────────────┘

AFTER (fixed):
┌─────────────────────────────────┐
│ setCurrentTotalXp(...)          │  Separate, clean update
└─────────────────────────────────┘
┌─────────────────────────────────┐
│ setAchievements((prev) => {     │
│   return filtered array         │  Pure function - no side effects
│ })                              │
└─────────────────────────────────┘
```

### The Fix

```javascript
const handleAchievementDeleted = useCallback((id) => {
  // Step 1: Get the points BEFORE any state changes
  const deleted = achievements.find((a) => a.id === id);
  const pointsToRemove = deleted?.points ?? 0;

  // Step 2: Update XP (separate operation)
  if (pointsToRemove > 0) {
    setCurrentTotalXp((xp) => xp - pointsToRemove);
  }

  // Step 3: Update achievements (pure function - no side effects)
  setAchievements((prev) => prev.filter((a) => a.id !== id));
}, [achievements]);  // ← Now has proper dependencies
```

### Key Changes

1. **Moved the side effect outside** - `setCurrentTotalXp` is now a separate call, not nested inside `setAchievements`
2. **Added proper dependencies** - `[achievements]` ensures the callback sees current data
3. **Pure setState callback** - `setAchievements` callback now only filters and returns

---

## Rules to Remember

1. **setState callbacks should be pure** - only compute and return the new value
2. **Don't nest setState calls** - keep them separate and sequential
3. **Always include dependencies** - if your callback uses external values, list them in the dependency array
4. **Empty `[]` = stale closures** - the function will forever see initial values

---

## Official React Documentation

- [useState and the `prev` pattern](https://react.dev/reference/react/useState#updating-state-based-on-the-previous-state)
- [useCallback and dependency arrays](https://react.dev/reference/react/useCallback)
- [Why state updates should be pure](https://react.dev/learn/keeping-components-pure)
- [How React batches state updates](https://react.dev/learn/queueing-a-series-of-state-updates)
