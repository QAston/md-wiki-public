# windows terminal

- <https://github.com/microsoft/terminal>
- <https://www.microsoft.com/en-us/p/windows-terminal-preview/9n0dx20hk701?activetab=pivot:overviewtab>
    # context menu implemented as a dll
    - https://github.com/microsoft/terminal/blob/049e37e514a50f5c2e95f1d8a883c7810169aba2/src/cascadia/ShellExtension/OpenTerminalHere.cpp
- [great "run wt here" menu](https://github.com/lextm/windowsterminal-shell)
- [disabling context menu](https://github.com/microsoft/terminal/issues/8105#issuecomment-726789079)
- you can install preview and uninstall regular version and wt.exe will then start preview

## shortcuts

* make sure that terminal doesn't define any shortcuts for input to the console, with exception of copy paste, so that the console applications can handle the keyboard input themselves
    * scrolling is fine because terminal history scrolling is a function of the 
    * ctrl-shift-c/ctrl-shift-v - terminal copy/paste
    * ctrl-c/ctrl-v - application copy/paste, unbound in terminal
* ctrl-shift-p - command panel
    * command panel can run wt.exe commands: <https://docs.microsoft.com/en-gb/windows/terminal/command-line-arguments?tabs=windows>
* ctrl+shift+f - find
* tab management:
    * ctrl+shift+t - new tab (default profile)
    * ctrl+shift+d - duplicate tab (use the same profile as focus)
    * ctrl+shift+`<num>` - new tab with profile 
    * ctrl+shift+space - new tab with profile (dropdown)
    * ctrl+tab - switch to next tab
    * ctrl+shift+tab - switch to prev tab
    * ctrl+alt+`<num>` - switch to tab
* pane management
    * closing a tab closes all panes in a tab
    * clicking open tab profile while holding alt will open a pane instead of tab
    * alt+shift+d - duplicate pane (use the same profile as focus)
    * alt+shift+w - close pane/tab when no panes left
    * alt+shift+arrow - resize pane
    * alt+shift+plus-split vertical (same profile as focus)
    * alt+shift+minus-split horizontal (same profile as focus)
    * alt+arrow - move focus
    * alt+shift+left - focus to last used pane
* window management
    * ctrl+shift+n - new window
    * alt+f4 - close window
    * f11 - fullscreen
    * alt+f11 - toggle always on top
* scrolling - standard keybindings
    scrollDown - "ctrl+down"
    scrollDownPage - "ctrl+pgdn"
    scrollUp - "ctrl+up"
    scrollUpPage - "ctrl+pgup"
    scrollToTop - "ctrl+home"
    scrollToBottom - "ctrl+end"

### config example:

```
{
    "$schema": "https://aka.ms/terminal-profiles-schema",
    "defaultProfile": "{574e775e-4f2a-5b96-ac1e-a2962a402336}",

    // You can add more global application settings here.
    // To learn more about global settings, visit https://aka.ms/terminal-global-settings

    // If enabled, selections are automatically copied to your clipboard.
    "copyOnSelect": false,

    // If enabled, formatted data is also copied to your clipboard
    "copyFormatting": false,

    "multiLinePasteWarning": false,

    "windowingBehavior": "useExisting",
    
    // A profile specifies a command to execute paired with information about how it should look and feel.
    // Each one of them will appear in the 'New Tab' dropdown,
    //   and can be invoked from the commandline with `wt.exe -p xxx`
    // To learn more about profiles, visit https://aka.ms/terminal-profile-settings
    "profiles":
    {
        "defaults":
        {
            // Put settings here that you want to apply to all profiles.
            "fontFace": "Delugia Mono Nerd Font",
            //"altGrAliasing": false,
        },
        "list":
        [
            {
                // Make changes here to the cmd.exe profile
                "guid": "{0caa0dad-35be-5f56-a8ff-afceeeaa6101}",
                "name": "cmd",
                "commandline": "cmd.exe",
                "hidden": false
            },
            {
                "guid": "{574e775e-4f2a-5b96-ac1e-a2962a402336}",
                "hidden": false,
                "name": "PowerShell",
                "source": "Windows.Terminal.PowershellCore"
            },
            {
                "guid": "{ac73a22d-256b-58dd-8d16-749c37c3aeaa}",
                "hidden": false,
                "name": "Artix",
                "commandline": "wsl.exe -d Artix"
            },
            {
                "guid": "{17da3cac-b318-431e-8a3e-7fcdefe6d114}",
                "name": "MINGW64 / MSYS2",
                "commandline": "C:/portable/msys/msys2_shell.cmd -defterm -where %__CD__% -no-start -mingw64",
                "icon": "C:/portable/msys/mingw64.ico"
              },
              {
                "guid": "{2d51fdc4-a03b-4efe-81bc-722b7f6f3820}",
                "name": "MINGW64-WINPATH / MSYS2",
                "commandline": "C:/portable/msys/msys2_shell.cmd -defterm -here -no-start -mingw64 -full-path",
                "icon": "C:/portable/msys/mingw32.ico"
              },
              {
                "guid": "{71160544-14d8-4194-af25-d05feeac7233}",
                "name": "MSYS / MSYS2",
                "commandline": "C:/portable/msys/msys2_shell.cmd -defterm -here -no-start -msys",
                "icon": "C:/portable/msys/msys2.ico"
              },
            {
                // Make changes here to the powershell.exe profile
                "guid": "{61c54bbd-c2c6-5271-96e7-009a87ff44bf}",
                "name": "Windows PowerShell",
                "commandline": "powershell.exe",
                "hidden": false
            },
            {
                "guid": "{b453ae62-4e3d-5e58-b989-0a998ec441b8}",
                "hidden": false,
                "name": "Azure Cloud Shell",
                "source": "Windows.Terminal.Azure"
            }
        ]
    },

    // Add custom color schemes to this array.
    // To learn more about color schemes, visit https://aka.ms/terminal-color-schemes
    "schemes": [],

    // Add custom keybindings to this array.
    // To unbind a key combination from your defaults.json, set the command to "unbound".
    // To learn more about keybindings, visit https://aka.ms/terminal-keybindings
    "keybindings":
    [
        // scrollback config - same as my readline and vscode configs
        { "command": "scrollDown", "keys": "ctrl+down" },
        { "command": "scrollDownPage", "keys": "ctrl+pgdn" },
        { "command": "scrollUp", "keys": "ctrl+up" },
        { "command": "scrollUpPage", "keys": "ctrl+pgup" },
        { "command": "scrollToTop", "keys": "ctrl+home" },
        { "command": "scrollToBottom", "keys": "ctrl+end" },
        {
            "command" : null, "keys" : ["ctrl+insert"]
        },
        {
            "command" : null, "keys" : ["shift+insert"]
        },
        {
            "command" : null, "keys" : ["ctrl+-"]
        },
        {
            "command" : null, "keys" : ["ctrl+plus"]
        },
        {
            "command" : null, "keys" : ["ctrl+0"]
        },
        {
            "command" : null, "keys" : ["ctrl+,"]
        },
        {
            "command" : null, "keys" : ["ctrl+shift+,"]
        },
        { "command": "toggleAlwaysOnTop", "keys": ["alt+f11"] },
        
        { "command": "closePane", "keys": "alt+shift+w" },
        { "command": { "action": "moveFocus", "direction": "previous" }, "keys": "alt+shift+left" },



        { "command": null, "keys": "ctrl+shift+w" },
        { "command": null, "keys": "ctrl+alt+left" },
        { "command": { "action": "splitPane", "split": "horizontal", "splitMode": "duplicate" }, "keys": "alt+shift+-" },
        { "command": { "action": "splitPane", "split": "vertical", "splitMode": "duplicate"}, "keys": "alt+shift+plus" },

        // Press Ctrl+Shift+F to open the search box
        { "command": "find", "keys": "ctrl+shift+f" },

        // Press Alt+Shift+D to open a new pane.
        // - "split": "auto" makes this pane open in the direction that provides the most surface area.
        // - "splitMode": "duplicate" makes the new pane use the focused pane's profile.
        // To learn more about panes, visit https://aka.ms/terminal-panes
        { "command": { "action": "splitPane", "split": "auto", "splitMode": "duplicate" }, "keys": "alt+shift+d" }
    ]
}
```