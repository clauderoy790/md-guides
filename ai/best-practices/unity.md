# Unity AI Development Best Practices

Guidelines for AI agents working on Unity projects. These rules prevent common issues where AI-generated files conflict with Unity's asset management system.

---

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
