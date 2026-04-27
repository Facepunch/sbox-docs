---
title: "Tools Addon: CodeEditors"
icon: "💻"
sources:
  - game/addons/tools/Code/CodeEditors/
created: 2026-04-27
updated: 2026-04-27
---

# CodeEditors (IDE Integration)

The `CodeEditors` directory in the Tools addon handles the integration between the s&box Editor and external Integrated Development Environments (IDEs). This is what enables you to double-click a script in the Asset Browser and have it automatically open in your preferred editor.

## Quick Working Example

You can implement custom editor integrations by inheriting from the `ICodeEditor` interface. The engine will automatically discover it and present it as an option in the Editor Preferences.

```csharp
using Editor.CodeEditors;
using Editor;

[Title( "My Custom IDE" )]
public class MyCustomIDE : ICodeEditor
{
    public void OpenFile( string path, int? line, int? column )
    {
        // Example: launch an external process passing the file path
        // MyCustomIDE.exe "path/to/file.cs" --line 15
        var args = $"\"{path}\"";
        if ( line.HasValue ) args += $" --line {line.Value}";

        var startInfo = new System.Diagnostics.ProcessStartInfo
        {
            CreateNoWindow = true,
            Arguments = args,
            FileName = "C:/Path/To/MyCustomIDE.exe"
        };
        System.Diagnostics.Process.Start( startInfo );
    }

    public void OpenSolution()
    {
        // Logic to open the entire .sln
    }

    public void OpenAddon( Project addon )
    {
        // Logic to open a specific addon project
    }

    public bool IsInstalled()
    {
        // Check registry or standard install paths
        return System.IO.File.Exists( "C:/Path/To/MyCustomIDE.exe" );
    }
}
```

1.  **Selecting an Editor:** Developers do not interact with this code directly; they use the **Editor -> Preferences -> General -> Code Editor** dropdown to select their IDE.
2.  **Visual Studio Integration:** The `VisualStudio` implementation uses `vswhere.exe` to reliably locate the installation path of the latest Visual Studio version that includes the .NET SDK, and uses `vsopen.exe` to seamlessly open files in a currently running instance of VS without spawning a new window.
3.  **Rider / VS Code:** Similar implementations exist for JetBrains Rider and Visual Studio Code, tailoring the command-line arguments to fit those specific applications.

## Troubleshooting

:::warning Common Gotchas
- **"Code Editor Not Found":** If your selected code editor isn't opening when you click a script, ensure that the editor is actually installed, and that it was installed in the default directory. The `IsInstalled()` checks often look in standard program file paths or use specific tools like `vswhere`.
- **Opening at the wrong line:** If you click an error in the s&box console and your IDE opens at the top of the file instead of the error line, the specific `ICodeEditor` implementation might not be passing the `line` or `column` parameters correctly in its `OpenFile` method.
:::

## Related Pages
- [Tools Addon Code Architecture](tools-code.md)

