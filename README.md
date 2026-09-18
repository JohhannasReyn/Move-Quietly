# Move Conflicts Quietly

Move Conflicts Quietly removes the repetitive filename-conflict decision from
Windows File Explorer copy and move operations.

When an incoming item has the same name as an item that already exists in the
destination, the mod keeps both by giving the incoming item the next free
`name (n).ext` style name instead of showing the **Replace or skip files**
prompt.

For example, a collision such as:

```
report.txt
```

will normally produce a unique name similar to:

```
report (2).txt
```

Further collisions take the next free number: `report (3).txt`, and so on. An
incoming `report (2).txt` that collides becomes `report (3).txt`, not
`report (2) (2).txt`.

## What it affects

The mod targets File Explorer (`explorer.exe`) and applies automatic
rename-on-collision behavior to copy and move operations performed through the
Windows `IFileOperation` interface, including common Explorer operations such
as copy/paste, cut/paste, and drag-and-drop moves.

The mod does **not** intentionally suppress unrelated error or security UI.
Permission errors, UAC/elevation prompts, disk errors, and other exceptional
conditions are still handled by Windows normally.

## How it works

Explorer uses the Windows Shell `IFileOperation` COM interface for many file
operations. This mod hooks the copy and move entry points. For every queued
item it resolves the destination path, checks whether the name is already
taken (on disk, or by another item queued in the same operation), and if so
passes an explicit `name (n).ext` name to the Shell.

As a safety net it also adds `FOF_RENAMEONCOLLISION` and
`FOFX_PRESERVEFILEEXTENSIONS` to the operation, so that even when the mod
cannot pick a name itself (a non-file-system destination, or a name that is
taken between the check and the copy) the Shell still keeps both instead of
prompting. In that fallback case the Shell chooses the name, which is its
usual `name - Copy.ext` pattern.

Existing operation flags are retained. If the caller did not explicitly set
operation flags, the documented `IFileOperation` defaults are preserved before
the collision flags are added.

## Important behavior and limitations

- Copying an item into its own folder keeps Explorer's normal
  `name - Copy.ext` behavior; the mod only renames items arriving from a
  different folder.
- Items that are not file-system objects (for example dragged out of a zip
  folder or a phone) fall back to the Shell's own collision naming.
- Rename-on-collision applies to Shell items, which can include folders as well
  as files. A same-name folder encountered during a copy or move may therefore
  be renamed instead of following Explorer's normal folder-merge conflict path.
- This version targets 64-bit File Explorer.
- Rename-only and delete operations are not intentionally modified.

## Testing

Before relying on the mod with important data, test it with disposable files
and folders. Useful cases include:

1. Copy one file into a folder that already contains a file with the same name.
2. Repeat the copy several times and verify that every copy is preserved.
3. Test cut/paste and drag-and-drop moves with a destination conflict.
4. Test a multi-file copy containing several conflicts.
5. Test a same-name folder collision so you understand the folder behavior on
   your Windows version.

## Source and issues

Source code and issue tracking:
https://github.com/JohhannasReyn/move-quietly

If you find a copy or move path that still displays Explorer's conflict chooser,
please include your Windows version, the exact operation used, and the Windhawk
log output in the issue report.
