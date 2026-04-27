---
title: "Tools Addon: LibraryManager"
icon: "inventory_2"
sources:
  - game/addons/tools/Code/LibraryManager/
created: 2026-04-27
updated: 2026-04-27
---

# Library Manager

The `LibraryManager` directory in the Tools addon implements the UI and logic for discovering, downloading, and managing addon dependencies within the s&box Editor.

## Quick Working Example

You can open the Library Manager programmatically from another editor script if you need to prompt the user to install a specific dependency:

```csharp
using Editor;

public class LibraryHelper
{
    [Menu( "Editor", "My Tools/Open Library Manager" )]
    public static void OpenManager()
    {
        // Finds or creates the Library Manager dock window
        EditorUtility.OpenWindow( "Library Manager" );
    }
}
```

1.  **Installing Dependencies:** When a user clicks "Install" on a library in the `LibraryDetail` view, the tool modifies the current project's `.sbproj` file to include the new dependency. The engine then automatically downloads the package contents in the background.
2.  **Asset Browsing:** You can search for packages by tags (e.g., "shader", "materials") using the search bar and facet dropdowns built into the `LibraryManagerDock`.

## Troubleshooting

:::warning Common Gotchas
- **UI State Desync:** If an addon is deleted directly from the file system instead of through the Library Manager, the manager might still show it as "Installed" until the project file is re-parsed or the editor is restarted.
- **Network Issues:** The `LibraryList` relies on querying the sbox.game backend API. If the editor is offline, the "Available" tab will fail to load or show cached results.
:::

## Related Pages
- [Tools Addon Code Architecture](tools-code.md)
- [Addons & Extensions](../index.md)

