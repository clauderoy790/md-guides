# Event Handling: preventDefault() vs stopPropagation()

Understanding how to properly control event behavior in JavaScript/React.

## Event Flow in the Browser

When you click on an element, the browser does two things:

1. **Event Propagation**: The click event "bubbles up" through the DOM tree (from the clicked element → parent → grandparent → etc.)
2. **Default Behavior**: Some elements have built-in behaviors (like `<a>` tags navigating to their href)

## What Each Method Does

### `e.stopPropagation()`
- Stops the event from bubbling up to parent elements
- Prevents parent click handlers from firing
- Does NOT stop the element's own default behavior

### `e.preventDefault()`
- Stops the element's default behavior
- For `<a>` tags: prevents navigation
- For `<form>` tags: prevents form submission
- For checkboxes: prevents toggling
- Does NOT stop event bubbling

## Real-World Example

Consider a clickable card with an interactive button inside:

```tsx
<a href="..." onClick={handleCardClick}>     ← Has default behavior (navigate)
  <button onClick={handleButtonClick}>       ← Your interactive button
    Click me
  </button>
</a>
```

When you click the button:
1. **Without `preventDefault()`**: The `<a>` tag's default navigation happens → link opens
2. **Without `stopPropagation()`**: The click bubbles to the `<a>` tag's onClick handler → card handler runs

**Solution**: Use both in the button handler:

```tsx
<button
  onClick={(e) => {
    e.preventDefault();      // Stop the <a> from navigating
    e.stopPropagation();     // Stop the click from reaching card handler
    handleButtonClick();
  }}
>
  Click me
</button>
```

## Common Use Cases

### Use `preventDefault()` when:
- Handling form submissions yourself:
  ```tsx
  <form onSubmit={e => { e.preventDefault(); handleSubmit(); }}>
  ```
- Custom link behavior:
  ```tsx
  <a onClick={e => { e.preventDefault(); customAction(); }}>
  ```
- Preventing default browser actions (text selection, right-click menus, etc.)

### Use `stopPropagation()` when:
- A child element shouldn't trigger parent handlers
- Buttons inside clickable cards (favorite hearts, delete buttons, etc.)
- Nested interactive elements where you want to isolate behavior

### Use both when:
- You need to prevent BOTH the default action AND parent handlers
- Interactive elements nested inside clickable containers
- Buttons/inputs inside `<a>` tags or other clickable wrappers

## Quick Reference

| Method | Stops Default Behavior | Stops Event Bubbling |
|--------|------------------------|---------------------|
| `preventDefault()` | ✅ Yes | ❌ No |
| `stopPropagation()` | ❌ No | ✅ Yes |

## Key Takeaway

- **`preventDefault()`** = "Don't do what you normally do"
- **`stopPropagation()`** = "Don't tell your parents about this"

Use both when you have interactive elements inside other interactive elements!
