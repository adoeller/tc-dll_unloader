# TC Plugin Unloader

TC Plugin Unloader lets external tools prepare updates, replacements, renames,
or deletion of Total Commander plugin files while Total Commander is running.

The program watches the plugin files configured in `wincmd.ini`. When another
process requests write, delete, rename, or replace access, TC Plugin Unloader:

1. notices the request with a shared Read/Handle (RH) opportunistic lock;
2. asks every detected Total Commander instance to unload the affected plugin
   category;
3. records that the operation may need a retry; and
4. watches the replacement file again after an atomic rename or replace.

Ordinary read access does not cause plugins to be unloaded.

RH monitoring is notification-only: it does not stop the original file
operation. If the first replacement fails because Total Commander still holds
the DLL, repeat the copy after the log reports the release request. The
category buttons in the GUI are the reliable way to prepare a replacement
*before* starting a copy.

## Requirements

- Windows 32-bit or 64-bit
- Total Commander 10.50 or later
- A local file system that supports Windows opportunistic locks, normally NTFS

Use the 64-bit release on a 64-bit version of Windows and the 32-bit release on
a 32-bit version of Windows. Either release can communicate with both 32-bit
and 64-bit Total Commander instances.

If the plugin directory is protected, for example below `C:\Program Files`, run
TC Plugin Unloader with sufficient permissions. It may also need to run at the
same elevation level as Total Commander and the program updating the plugin.

## Plugin category masks

The console options and GUI buttons use these Total Commander plugin masks:

| Mask | Type | Description |
|---:|---|---|
| `1` | WCX | Packer plugins |
| `2` | WFX | File-system plugins |
| `4` | WLX | Lister plugins |
| `8` | WDX | Content plugins |
| `16` | TCMatch | `tcmatch.dll` or `tcmatch64.dll` |
| `0` | All | All supported plugin categories |

Masks can be combined by adding them. For example, `12` unloads WLX and WDX
plugins (`4 + 8`). Valid mask values are `0` through `31`.

# Console application

The console program is `TCPluginUnloader.exe`. Its interface and messages are
always in English.

Open Command Prompt or PowerShell in the directory containing the executable.

Running the console application without arguments displays its English command
help and exits without loading the Total Commander configuration:

```text
TCPluginUnloader.exe
```

## Monitor all configured plugins

```text
TCPluginUnloader.exe --monitor
```

The program finds the Total Commander installation and `wincmd.ini` through
the standard Ghisler registry entries. If `UseIniInProgramDir` is enabled, the
`wincmd.ini` beside Total Commander takes precedence.

Press Enter to stop monitoring.

To use a specific INI file:

```text
TCPluginUnloader.exe --ini="C:\Program Files\totalcmd\wincmd.ini" --monitor
```

The following sections are read, including supported `RedirectSection` files:

- `[PackerPlugins]`
- `[FileSystemPlugins]`
- `[ListerPlugins]`
- `[ContentPlugins]`
- `MatchFileName` in `[Configuration]`

The `%COMMANDER_PATH%`, `%COMMANDER_INI%`, and Windows environment variables in
plugin paths are expanded automatically.

For 64-bit Total Commander, a configured unsuffixed plugin such as
`example.wlx` may be loaded from `example.wlx64`. If the configured file does
not exist but its `64`-suffixed variant does, only the existing 64-bit file is
monitored and the missing unsuffixed file is not reported as an error. If both
files exist, both are monitored.

## List the detected plugin files

Supply an INI file without `--monitor` to print its configured plugin files:

```text
TCPluginUnloader.exe --ini="D:\TotalCmd\wincmd.ini"
```

Each output line starts with the plugin category mask followed by the full
plugin path.

## Monitor one plugin file

```text
TCPluginUnloader.exe --watch="C:\Plugins\Example\example.wlx64" --mask=4
```

`--mask` defaults to `4` (WLX) when omitted.

To stop automatically after a specified number of seconds:

```text
TCPluginUnloader.exe --watch="C:\Plugins\Example\example.wlx64" --mask=4 --duration=60
```

Use `--observe` to log RH events without sending an unload command. Repeated
events for a category are coalesced for one second by default; adjust that with
`--debounce-ms=<milliseconds>`.

## Unload plugins immediately

This sends one unload command without starting the monitor:

```text
TCPluginUnloader.exe --unload=4
```

For example, use `--unload=12` for WLX and WDX plugins or `--unload=0` for all
plugin categories. The command is sent to every detected Total Commander
instance.

## List Total Commander instances

```text
TCPluginUnloader.exe --list-tc
```

The output contains the process ID, window handle, and executable path of every
verified Total Commander instance.

## Advanced command ID override

The default internal Total Commander command ID is `2913`. Override it only if
required by the Total Commander version being used:

```text
TCPluginUnloader.exe --command-id=2913 --unload=4
```

The same option can be combined with monitoring commands.

## Console exit codes

| Code | Meaning |
|---:|---|
| `0` | Completed successfully |
| `1` | Invalid option, missing INI, no plugin entries, or another error |
| `2` | An unload command was requested, but no Total Commander instance accepted it |

# GUI application

The graphical program is `TCPluginUnloaderGUI.exe`.

Keep the `languages` directory beside the executable:

```text
TCPluginUnloaderGUI.exe
languages\
  TCPluginUnloaderGUI.de.po
  TCPluginUnloaderGUI.uk.po
  TCPluginUnloaderGUI.ru.po
```

## First start

The GUI normally detects `wincmd.ini` automatically. Verify the displayed path
and change it with the browse button when using a portable or alternative Total
Commander installation.

The main controls are:

- **Start monitoring**: reloads the plugin list from `wincmd.ini` and protects
  the detected plugin files.
- **Stop**: stops all plugin monitoring.
- **WCX, WFX, WLX, WDX**: immediately unload that category in every detected
  Total Commander instance. Use them as **Prepare replacement** before copying
  a plugin. These are one-time actions, not toggle buttons.
- **Save settings**: saves the current paths and behavior options and applies
  the selected log-file path.

Hover over an unload button to see its category and action.

## Tray behavior

- **Minimize to tray** hides the main window and removes its taskbar button.
- **Close to tray** hides the window instead of terminating the program.
- Double-click the tray icon or choose **Show** to restore the window.
- Choose **Exit** from the tray menu to terminate the program when close-to-tray
  behavior is enabled.

The monitoring process continues while the main window is hidden.

## GUI languages

The GUI uses the Windows display language when a matching translation is
available:

- English, built in and used as the default fallback
- German
- Ukrainian
- Russian

To force a language for one start, use one of these commands:

```text
TCPluginUnloaderGUI.exe --lang=en
TCPluginUnloaderGUI.exe --lang=de
TCPluginUnloaderGUI.exe --lang=uk
TCPluginUnloaderGUI.exe --lang=ru
```

The diagnostic log remains in English so that console and GUI logs use the same
technical terminology.

## Settings and log files

By default, the GUI stores its settings beside the executable:

```text
TCPluginUnloaderGUI.exe
settings.ini
```

File logging is disabled until a log-file path is entered in the GUI and
**Save settings** is selected. The live log view remains available without a
log file.

The settings file contains the selected `wincmd.ini`, optional log path, tray
behavior, automatic monitoring option, unload command ID, and window position
and size.

For a separate or portable settings file, start the GUI with:

```text
TCPluginUnloaderGUI.exe --config="D:\Portable\TCPluginUnloader\settings.ini"
```

The log view follows new messages only while its cursor is already at the end.
Selecting text or scrolling upward keeps the current selection and position.

# Monitoring notes and troubleshooting

## Missing plugin directories

Old plugin entries can remain in `wincmd.ini` after a plugin has been removed.
For these entries, the program immediately logs one warning without retrying the
missing path. It continues checking silently in the background. If the
directory or plugin file reappears, monitoring starts automatically.

The three short retries are used only when a plugin file exists but the program
cannot establish an opportunistic lock. After the third failure, one warning is
logged and silent background checks continue.

Windows error `2` means that the configured file or directory does not exist.
Remove obsolete entries from `wincmd.ini` if they are no longer needed.

## A plugin cannot be updated

Check the following:

1. TC Plugin Unloader is still running and monitoring is active.
2. The correct `wincmd.ini` is selected.
3. The plugin appears in the detected list and its path is correct.
4. TC Plugin Unloader has permission to open the plugin file.
5. No open Lister, quick-view window, file-system connection, or other active
   operation is preventing Total Commander from unloading the plugin.

With the default RH monitor, a replacement is not paused. When the first copy
reports a sharing error, wait for the `RH release requested` log line and use
**Retry** in the copying application (or start the copy again). To avoid that
first failure, click the category button (WCX/WFX/WLX/WDX) before copying.

If a plugin is actively in use, Total Commander may be unable to unload it. In
that case Windows can still reject the external replacement because Total
Commander continues to hold the DLL.

## Rename and replace operations

Installers commonly create a temporary file and atomically rename it over the
old DLL. Directory notifications detect this operation and protect the new file
again. Changes to the plugin configuration itself are not reloaded
automatically; stop and restart monitoring after editing `wincmd.ini`.

## File-system limitations

The program is intended for local Windows file systems with opportunistic-lock
support. Network shares, compatibility layers, and non-NTFS file systems may
not provide the same notification behavior. RH monitoring cannot guarantee
that the first write is delayed; that requires a kernel minifilter.
