# Trash Tools

**uunlnk-style file protection & drop-in shell-script-based replacements for trash & unlink functionality in third-party macOS file managers**

⚠️ **Please note:** the Trash Tools shell scripts are **not meant for usage in macOS Finder**: you will need a **third-party file manager** like **[Nimble Commander](https://magnumbytes.com/)** that supports **direct shell script execution** and allows users to **remap system-default keyboard shortcuts**.

## Impetus
If you want to protect your files against accidental deletion in macOS, you only have the ability to **"lock"** files, as the Finder calls it, which in reality corresponds to the old Unix **immutable flag** (`uchg`), or you can use a `deny delete` **access control entry** (ACE). In both cases this also renders the file uneditable, which is why in the former case (`uchg`), when you want to modify a "locked" file, some applications create a copy of the "locked" file's contents, create a new file with a new inode number, and then delete the "locked" original, but very few applications actually respect the user's wishes and re-apply the immutable flag to the new file. Files protected with a `deny delete` ACE apparently can not be handled by the system APIs at all, and any proper modifications will fail.

There is, however, an old Unix flag called **system no-unlink** (`sunlnk`), which (also on macOS) protects important files from deletion, but still allows certain file modifications. The variant **user no-unlink** (`uunlnk`) is actually present in macOS, and can even be set with the right tools, but it is disregarded by the system and removed as soon as such a file is opened or modified. At any rate, the `sunlnk` and `uunlnk` flags would not be sufficient for our needs anyway, because they still block the user from performing certain modifications like mounting over a file or renaming it. Especially the latter is important for every-day file usage.

**Trash Tools**, in tandem with the `uunlnk` script, offer at least a workaround: it will add an **extended attribute** (XA) to a file, and if Trash Tool's file removal tools access that file, any trash or unlink operation will fail, while the user is still free to perform any and all other file operations, including renaming, moving to a different location, and modifying file contents. These protected files can still be trashed or unlinked by other processes, but if the user remaps the keyboard shortcuts of all trash and unlink routines in his file manager to the Trash Tools scripts, any **accidental file deletions** will at least be prevented in the file manager.

Most users will probably have no need for such a solution, because the normal user just moves files to the Trash, and can then easily restore the file, if it was an accident. But the tools might be valuable for those who perform a lot of permanent deletions on scratch or RAM disks.

### Notes
* file copies of protected files will have the same protection as the original
* on volumes without support for extended attributes, e.g. mounted NAS volumes, the protection flag and tag for `foo` will be written as `._foo`

## Requisites
* **`[trash](https://github.com/sindresorhus/macos-trash)`** (install e.g. with **[Homebrew](https://brew.sh/)**)

## Installation
* option #1: clone repository and create symbolic links into one of your bin directories
* option #2: download repository and copy the shell scripts into one of your bin directories
* if necessary, set the executable bits with `chmod +x`

## Setup
* find the `CFBundleID` in the `./Contents/Info.plist` of your default file manager (preferably *not* macOS Finder), e.g. `info.filesmanager.Files` for Nimble Commander
* execute the command `defaults read -g NSFileViewer 2>/dev/null`
* if there is no output, or if the bundle ID does not match your default file manager's bundle ID, execute the command `defaults write -g NSFileViewer <CFBundleID>`, while replacing "<CFBundleID>" with the bundle ID of your default file manager
* execute the command `tt-setup`
* populate the file protection list at `~/.config/TrashTools/protections.txt` for files that shall never be unlinked or moved to the Trash (**filenames only**)

## Examples for keyboard shortcuts
* `uunlnk` (toggle file protection): CMD-OPT-CTRL-K
* `trashes` (move to Trash): CMD-DEL
* `empty-trashes` (empty Trashes): CMD-SHIFT-DEL
* `empty-trashes --force` (empty Trashes without asking): CMD-OPT-SHIFT-DEL
* `undo-trashes` (undo trash operations): CMD-CTRL-DEL
* `unlink` (permanent file deletion): CMD-OPT-DEL

## Functionality
### `uunlnk`
Toggle file protection by writing the extended attribute `local.lcars.fpsr#PS`. Additionally, a user tag called "Protected" will be added, so you can quickly list all your protected files with Spotlight or `mdfind` and `list-protected`.

### `list-protected`
List currently protected files on all volumes (only possible with indexing enabled) or in specified paths. Examples:

* `list-protected` (raw `mdfind` list of all protected filepaths)
* `list-protected -v` (list protected files with extended attribute prefixed)
* `list-protected -v 2>/dev/null | sort -n -t ';' -r -k2` (sort protected files, most recent first)
* `list-protected -v 2>/dev/null | sort -n -t ';' -r -k2 | awk -F":" '{print substr($0, index($0,$2))}' | sort` (sort protected files, most recent first, filenames only & sorted by name)
* `list-protected <file>` (list protection status of a single non-directory file)
* `list-protected <directory>` (list protection status of a single directory file or volume, and of all its contents)

### `trashes`
Move files to the relevant Trash, including iCloud Trash. On volumes without `.Trash` or `.Trashes`, you will need to use the `unlink` command. Files without write access will be handled by macOS Finder; otherwise the requisite `trash` will call the regular system API.

Note: using the system API or the Finder (for files without write access) instead of simple `mv` routines is important, because the system's move-to-Trash functionality is important to properly remove certain filetypes, e.g. for an effective uninstallation of applications containing system or network extensions.

### `empty-trashes`
Empty all Trashes. Use the `--force` argument with an alternate keyboard shortcut, if you want to skip the prompt. The command supports all Trashes, including iCloud Trash, `/System/Volumes/.Trashes/501` and `./Trashes/501` on any mounted & writable volume.

Note: for the same reasons stated above, `empty-trashes` will still use the Finder for emptying the Trashes; if you have disabled Finder, `empty-trashes` will launch Finder hidden and in the background, and immediately quit Finder again, so the empty-Trashes operation will be barely noticeable.

### `undo-trashes`
⚠️ **Note: `undo-trashes` is not yet implemented** and needs some research & a lot of work first.

### `unlink`
Permanently delete (unlink) files. Internally, this command uses `rm -rf`. In case the executing user has no write access, files will be removed with `sudo -S rm -rf`, and the user needs to enter his administrator password first. ⚠️ As in any situation, please handle permanent file deletions with caution!

## Uninstall Trash Tools
* delete the repository and all copies or symbolic links of the following CLIs: `tt-setup` `uunlnk` `list-protected` `trashes` `empty-trashes` `undo-trashes` `unlink`
* `~/.config/TrashTools`
* `~/.cache/Finder`

## Screenshots

![sg01](https://github.com/JayBrown/Trash-Tools/blob/master/img/01_uunlnk_protected.png)

![sg02](https://github.com/JayBrown/Trash-Tools/blob/master/img/02_uunlnk_unprotected.png)

![sg03](https://github.com/JayBrown/Trash-Tools/blob/master/img/03_unlink_protected-error.png)

![sg04](https://github.com/JayBrown/Trash-Tools/blob/master/img/04_empty-trashes.png)

![sg05](https://github.com/JayBrown/Trash-Tools/blob/master/img/05_trashes_usingFinder.png)

![sg06](https://github.com/JayBrown/Trash-Tools/blob/master/img/06_unlink.png)

![sg07](https://github.com/JayBrown/Trash-Tools/blob/master/img/07_unlink_error.png)

![sg08](https://github.com/JayBrown/Trash-Tools/blob/master/img/08_unlink_root.png)

