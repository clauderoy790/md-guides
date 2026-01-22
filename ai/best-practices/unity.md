# Unity AI Development Best Practices

Guidelines for AI agents working on Unity projects. These rules prevent common issues where AI-generated files conflict with Unity's asset management system.

---

# Important: Unity version used in project
The versions that we are using is Unity 6.3 LTS. Ensure that the features that you are planning/implementing are compatible with this version. A lot of documentation on Unity is from old unity version.

## Core Principle

**Unity manages its own metadata.** Never create or modify files that Unity generates automatically (`.meta` files, GUIDs, serialized references).

---

## ScriptableObjects

### Never Do This
- Write `.asset` files directly (they contain GUIDs that Unity must generate)
- Create `.asset.meta` files
- Use placeholder GUIDs like `guid: SOME_GUID_HERE`

### Always Do This
Create **Editor scripts** that use Unity's AssetDatabase API:

```csharp
// Assets/Editor/DataSetup.cs
using UnityEditor;

[MenuItem("MyGame/Create Data")]
public static void CreateData()
{
    var asset = ScriptableObject.CreateInstance<MyDataType>();
    asset.someField = "value";

    AssetDatabase.CreateAsset(asset, "Assets/Data/MyAsset.asset");
    AssetDatabase.SaveAssets();
}
```

### Why?
- Unity generates correct GUIDs automatically
- Cross-references between ScriptableObjects work properly
- No "Missing Script" or reset values when opening in Editor

### For Bulk Data Creation
1. Create a factory class with helper methods for each ScriptableObject type
2. Create a setup script with a `[MenuItem]` that creates all initial data
3. Wire up cross-references after all assets exist
4. Tell the user to run the menu command in Unity

---

## Meta Files

### Never Do This
- Create `.meta` files manually
- Modify existing `.meta` files
- Include GUIDs in any file you write

### Always Do This
- Let Unity generate `.meta` files automatically when it imports assets
- If a `.meta` file is missing, Unity will regenerate it on next import

---

## Scene Files (.unity)

### Never Do This
- Write `.unity` files directly (complex YAML with internal references)
- Modify scene files outside of Unity

### Always Do This
- Describe scene setup in documentation or comments
- Create Editor scripts that build scenes programmatically if needed
- Tell the user what to create manually in the Scene view

---

## Prefabs (.prefab)

### Never Do This
- Write `.prefab` files directly

### Always Do This
- Describe prefab structure in comments/documentation
- Create prefabs via Editor scripts using `PrefabUtility.SaveAsPrefabAsset()`
- Or tell the user to create prefabs manually in Unity

---

## Singleton Pattern

### Naming
Every singleton needs to have a Manager suffix in the class/filename. For example: GameManager.cs

### Always Do This
```csharp
// Simple, clean, sufficient for Unity projects
public class Singleton<T> : MonoBehaviour where T : MonoBehaviour
{
    public static T Instance { get; private set; }

    protected virtual void Awake()
    {
        if (Instance == null)
        {
            Instance = this as T;

            // If we have a parent, mark the root parent instead
            // This allows organizing singletons under a "Managers" GameObject
            Transform root = transform.root;
            if (root != transform)
            {
                DontDestroyOnLoad(root.gameObject);
            }
            else
            {
                DontDestroyOnLoad(gameObject);
            }
        }
        else
        {
            Destroy(gameObject); // Destroy duplicates
        }
    }

    protected virtual void OnDestroy()
    {
        if (Instance == this)
        {
            Instance = null; // Clean up reference
        }
    }
}

// Usage
public class GameManager : Singleton<GameManager>
{
    protected override void Awake()
    {
        base.Awake();
        // Your initialization here
    }
}
```

**Hierarchy Example:**
```
Scene Hierarchy:
├── Main Camera
├── Directional Light
└── Managers (will be marked with DontDestroyOnLoad automatically)
    ├── GameManager
    ├── AudioManager
    └── SaveManager
```

### Why?
- **Unity is single-threaded** - No need for `lock()` or thread safety
- **Auto-creation hides problems** - Better to know if GameManager is missing from scene
- **Simpler = fewer bugs** - 60 lines → 15 lines, same functionality
- **OnDestroy cleanup** prevents rare edge cases with scene reloading

### What's Actually Useful
✅ **Duplicate prevention** - Destroys extras if multiple exist
✅ **DontDestroyOnLoad** - Persists across scenes
✅ **Instance cleanup** - Prevents dangling references
❌ Thread-safe locking - Overkill for Unity
❌ Auto-creation - Hides missing GameObjects
❌ Application quit flag - Minor benefit, adds complexity

---

## Input System

We should always be using the new Unity input system (com.unity.inputsystem)

### Never Do This (old input system)
```csharp
// Old Input Manager (deprecated)
if (Input.GetKeyDown(KeyCode.Space))
{
    Jump();
}

if (Input.GetButtonDown("Fire1"))
{
    Shoot();
}
```

### Always Do This (new input system)

## For simple test scripts
```csharp
using UnityEngine.InputSystem;


// New Input System - Simple approach for test/debug scripts
private void Update()
{
    if (Keyboard.current == null) return; // Null check for safety

    if (Keyboard.current.spaceKey.wasPressedThisFrame)
    {
        Jump();
    }

    if (Mouse.current.leftButton.wasPressedThisFrame)
    {
        Shoot();
    }
}
```

## For Production Code
Use Input Actions for rebindable controls:

```csharp
using UnityEngine.InputSystem;

public class PlayerController : MonoBehaviour
{
    [SerializeField] private InputActionAsset inputActions;
    private InputAction moveAction;
    private InputAction jumpAction;

    private void Awake()
    {
        moveAction = inputActions.FindActionMap("Player").FindAction("Move");
        jumpAction = inputActions.FindActionMap("Player").FindAction("Jump");
    }

    private void OnEnable()
    {
        moveAction.Enable();
        jumpAction.Enable();
        jumpAction.performed += OnJump;
    }

    private void OnDisable()
    {
        moveAction.Disable();
        jumpAction.Disable();
        jumpAction.performed -= OnJump;
    }

    private void Update()
    {
        Vector2 input = moveAction.ReadValue<Vector2>();
        Move(input);
    }

    private void OnJump(InputAction.CallbackContext context)
    {
        Jump();
    }
}
```

### Why?
- **New Input System is the standard** - Old Input Manager is deprecated
- **Mobile support** - Touch and gamepad input work the same way
- **Rebindable controls** - Players can change key bindings
- **Multi-device** - Keyboard, mouse, touch, gamepad unified API

### Quick Reference
| Old Input Manager | New Input System |
|------------------|------------------|
| `Input.GetKeyDown(KeyCode.A)` | `Keyboard.current.aKey.wasPressedThisFrame` |
| `Input.GetKey(KeyCode.A)` | `Keyboard.current.aKey.isPressed` |
| `Input.GetMouseButtonDown(0)` | `Mouse.current.leftButton.wasPressedThisFrame` |
| `Input.mousePosition` | `Mouse.current.position.ReadValue()` |
| `Input.GetAxis("Horizontal")` | Use Input Actions |

---

## Finding Objects in Scene

### Never Do This (deprecated)
```csharp
// Old API (deprecated in Unity 2023+)
MyComponent obj = FindObjectOfType<MyComponent>();
MyComponent obj = GameObject.FindObjectOfType<MyComponent>();
MyComponent[] objs = FindObjectsOfType<MyComponent>();
```

### Always Do This (new API)
```csharp
// New API - Use FindFirstObjectByType for single objects
MyComponent obj = FindFirstObjectByType<MyComponent>();

// For finding all objects of a type
MyComponent[] objs = FindObjectsByType<MyComponent>(FindObjectsSortMode.None);

// If you need sorted results (slower, but deterministic)
MyComponent[] objs = FindObjectsByType<MyComponent>(FindObjectsSortMode.InstanceID);
```

### Why?
- **FindObjectOfType is deprecated** - Unity 2023+ shows compiler warnings
- **New API is clearer** - Explicit about whether you want first or all objects
- **Better performance** - FindFirstObjectByType can stop searching after finding one
- **Sorting control** - FindObjectsByType lets you choose if results should be sorted

### Quick Reference
| Old API (deprecated) | New API |
|---------------------|---------|
| `FindObjectOfType<T>()` | `FindFirstObjectByType<T>()` |
| `FindObjectsOfType<T>()` | `FindObjectsByType<T>(FindObjectsSortMode.None)` |
| `GameObject.FindObjectOfType<T>()` | `FindFirstObjectByType<T>()` |

### When to Use Each
- **FindFirstObjectByType**: When you need any one instance (e.g., finding a singleton manager)
- **FindObjectsByType**: When you need all instances (e.g., finding all enemies in scene)

---

## C# Scripts

### Safe to Create
- `.cs` files (Unity generates the `.meta` automatically on import)
- Any code files

### Structure
```
Assets/
├── Scripts/           # C# scripts (safe to create)
├── Editor/            # Editor-only scripts (safe to create)
└── ScriptableObjects/ # Assets created via Editor scripts (NOT by AI)
```

---

## Project Settings

### Never Do This
- Modify files in `ProjectSettings/` folder directly
- Change `ProjectSettings.asset`, `QualitySettings.asset`, etc.

### Always Do This
- Document what settings to change
- Tell the user to modify via Edit > Project Settings in Unity

---

## When Planning Unity Projects

1. **Phase 2 (Data Models)** should include:
   - Creating ScriptableObject C# classes (safe)
   - Creating Editor scripts to generate asset instances (safe)
   - NOT creating `.asset` files directly

2. **Tell the user** to run setup scripts after the phase:
   ```
   After this phase, open Unity and run:
   Menu > MyGame > Setup All Data
   ```

3. **Verify in manual testing** that assets were created correctly

---

## Quick Reference

| File Type | AI Can Create? | Method |
|-----------|---------------|--------|
| `.cs` | Yes | Write directly |
| `.asset` | No | Use Editor script + AssetDatabase |
| `.meta` | No | Unity generates automatically |
| `.unity` | No | Describe setup, or use Editor script |
| `.prefab` | No | Use PrefabUtility in Editor script |
| `ProjectSettings/*` | No | Document changes for user |

---

## Example Editor Script Pattern

```csharp
using UnityEngine;
using UnityEditor;

public static class MyGameDataSetup
{
    [MenuItem("MyGame/Setup All Data")]
    public static void SetupAllData()
    {
        // 1. Create assets with no dependencies first
        var itemA = CreateItem("Item A", 100);
        var itemB = CreateItem("Item B", 200);

        // 2. Create assets that reference others
        var collection = ScriptableObject.CreateInstance<ItemCollection>();
        collection.items = new[] { itemA, itemB };
        AssetDatabase.CreateAsset(collection, "Assets/Data/AllItems.asset");

        // 3. Save everything
        AssetDatabase.SaveAssets();
        AssetDatabase.Refresh();

        Debug.Log("Data setup complete!");
    }

    private static ItemData CreateItem(string name, int value)
    {
        var item = ScriptableObject.CreateInstance<ItemData>();
        item.itemName = name;
        item.value = value;
        AssetDatabase.CreateAsset(item, $"Assets/Data/Items/{name}.asset");
        return item;
    }
}
```

---

## Troubleshooting

### "Missing Script" on ScriptableObjects
- The `.asset` file references a script GUID that doesn't match
- Solution: Delete the `.asset` file, recreate via Editor script

### Values Reset When Opening Unity
- GUIDs in the `.asset` file don't match actual script GUIDs
- Solution: Recreate assets using AssetDatabase API

### Cross-references are Null
- Assets were created in wrong order, or references weren't saved
- Solution: Create all assets first, wire up references after, call `EditorUtility.SetDirty()` and `AssetDatabase.SaveAssets()`
