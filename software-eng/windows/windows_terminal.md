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
    "actions": 
    [
        {
            "command": 
            {
                "action": "scrollDown"
            },
            "keys": "ctrl+down"
        },
        {
            "command": "scrollDownPage",
            "keys": "ctrl+pgdn"
        },
        {
            "command": 
            {
                "action": "scrollUp"
            },
            "keys": "ctrl+up"
        },
        {
            "command": "scrollToBottom",
            "keys": "ctrl+end"
        },
        {
            "command": "scrollUpPage",
            "keys": "ctrl+pgup"
        },
        {
            "command": "scrollToTop",
            "keys": "ctrl+home"
        },
        {
            "command": "unbound",
            "keys": "ctrl+insert"
        },
        {
            "command": "unbound",
            "keys": "shift+insert"
        },
        {
            "command": "unbound",
            "keys": "ctrl+-"
        },
        {
            "command": "unbound",
            "keys": "ctrl+plus"
        },
        {
            "command": "unbound",
            "keys": "ctrl+0"
        },
        {
            "command": "unbound",
            "keys": "ctrl+,"
        },
        {
            "command": "unbound",
            "keys": "ctrl+shift+,"
        },
        {
            "command": "unbound",
            "keys": "alt+enter"
        },
        {
            "command": "unbound",
            "keys": "ctrl+shift+w"
        },
        {
            "command": "unbound",
            "keys": "ctrl+alt+left"
        },
        {
            "command": "toggleAlwaysOnTop",
            "keys": "alt+f11"
        },
        {
            "command": "closePane",
            "keys": "alt+shift+w"
        },
        {
            "command": 
            {
                "action": "moveFocus",
                "direction": "previous"
            },
            "keys": "alt+shift+left"
        },
        {
            "command": 
            {
                "action": "splitPane",
                "split": "horizontal",
                "splitMode": "duplicate"
            },
            "keys": "alt+shift+-"
        },
        {
            "command": 
            {
                "action": "splitPane",
                "split": "vertical",
                "splitMode": "duplicate"
            },
            "keys": "alt+shift+plus"
        },
        {
            "command": "find",
            "keys": "ctrl+shift+f"
        },
        {
            "command": 
            {
                "action": "splitPane",
                "split": "auto",
                "splitMode": "duplicate"
            },
            "keys": "alt+shift+d"
        }
    ],
    "copyFormatting": "none",
    "copyOnSelect": false,
    "defaultProfile": "{574e775e-4f2a-5b96-ac1e-a2962a402336}",
    "multiLinePasteWarning": false,
    "profiles": 
    {
        "defaults": {},
        "list": 
        [
            {
                "commandline": "cmd.exe",
                "fontFace": "Delugia Nerd Font",
                "guid": "{0caa0dad-35be-5f56-a8ff-afceeeaa6101}",
                "hidden": false,
                "name": "cmd"
            },
            {
                "fontFace": "Delugia Mono Nerd Font",
                "guid": "{574e775e-4f2a-5b96-ac1e-a2962a402336}",
                "hidden": false,
                "name": "PowerShell",
                "source": "Windows.Terminal.PowershellCore"
            },
            {
                "commandline": "wsl.exe -d Artix",
                "fontFace": "Delugia Mono Nerd Font",
                "guid": "{ac73a22d-256b-58dd-8d16-749c37c3aeaa}",
                "hidden": false,
                "icon": "F:\\Artix\\Artix.ico",
                "name": "Artix",
                "source": "Windows.Terminal.Wsl"
            },
            {
                "commandline": "powershell.exe",
                "fontFace": "Delugia Mono Nerd Font",
                "guid": "{61c54bbd-c2c6-5271-96e7-009a87ff44bf}",
                "hidden": false,
                "name": "Windows PowerShell"
            },
            {
                "commandline": "C:/portable/msys/msys2_shell.cmd -defterm -here -no-start -mingw64",
                "fontFace": "Delugia Mono Nerd Font",
                "guid": "{17da3cac-b318-431e-8a3e-7fcdefe6d114}",
                "icon": "C:/portable/msys/mingw64.ico",
                "name": "MINGW64 / MSYS2"
            },
            {
                "commandline": "C:/portable/msys/msys2_shell.cmd -defterm -here -no-start -mingw64 -full-path",
                "fontFace": "Delugia Mono Nerd Font",
                "guid": "{2d51fdc4-a03b-4efe-81bc-722b7f6f3820}",
                "icon": "C:/portable/msys/mingw32.ico",
                "name": "MINGW64-WINPATH / MSYS2"
            },
            {
                "commandline": "C:/portable/msys/msys2_shell.cmd -defterm -here -no-start -msys",
                "fontFace": "Delugia Mono Nerd Font",
                "guid": "{71160544-14d8-4194-af25-d05feeac7233}",
                "icon": "C:/portable/msys/msys2.ico",
                "name": "MSYS / MSYS2"
            },
            {
                "guid": "{581f6350-7f18-5b58-96d2-e8600fa2c88d}",
                "hidden": true,
                "name": "DriveArch",
                "source": "Windows.Terminal.Wsl"
            },
            {
                "guid": "{b453ae62-4e3d-5e58-b989-0a998ec441b8}",
                "hidden": false,
                "name": "Azure Cloud Shell",
                "source": "Windows.Terminal.Azure"
            },
            {
                "fontFace": "Delugia Mono Nerd Font",
                "guid": "{a5a97cb8-8961-5535-816d-772efe0c6a3f}",
                "hidden": false,
                "icon": "F:\\Arch\\Arch.ico",
                "name": "Arch",
                "source": "Windows.Terminal.Wsl"
            },
            {
                "guid": "{674ddf39-3f49-501f-ac73-e5f68f362f89}",
                "hidden": false,
                "name": "DockerArtix",
                "source": "Windows.Terminal.Wsl"
            },
            {
                "guid": "{6e8bfb28-6f9b-5298-8f18-9b3f5fad9ee8}",
                "hidden": false,
                "name": "DockerArch",
                "source": "Windows.Terminal.Wsl"
            }
        ]
    },
    "schemes": 
    [
        {
            "background": "#0C0C0C",
            "black": "#0C0C0C",
            "blue": "#0037DA",
            "brightBlack": "#767676",
            "brightBlue": "#3B78FF",
            "brightCyan": "#61D6D6",
            "brightGreen": "#16C60C",
            "brightPurple": "#B4009E",
            "brightRed": "#E74856",
            "brightWhite": "#F2F2F2",
            "brightYellow": "#F9F1A5",
            "cursorColor": "#FFFFFF",
            "cyan": "#3A96DD",
            "foreground": "#CCCCCC",
            "green": "#13A10E",
            "name": "Campbell",
            "purple": "#881798",
            "red": "#C50F1F",
            "selectionBackground": "#FFFFFF",
            "white": "#CCCCCC",
            "yellow": "#C19C00"
        },
        {
            "background": "#012456",
            "black": "#0C0C0C",
            "blue": "#0037DA",
            "brightBlack": "#767676",
            "brightBlue": "#3B78FF",
            "brightCyan": "#61D6D6",
            "brightGreen": "#16C60C",
            "brightPurple": "#B4009E",
            "brightRed": "#E74856",
            "brightWhite": "#F2F2F2",
            "brightYellow": "#F9F1A5",
            "cursorColor": "#FFFFFF",
            "cyan": "#3A96DD",
            "foreground": "#CCCCCC",
            "green": "#13A10E",
            "name": "Campbell Powershell",
            "purple": "#881798",
            "red": "#C50F1F",
            "selectionBackground": "#FFFFFF",
            "white": "#CCCCCC",
            "yellow": "#C19C00"
        },
        {
            "background": "#282C34",
            "black": "#282C34",
            "blue": "#61AFEF",
            "brightBlack": "#5A6374",
            "brightBlue": "#61AFEF",
            "brightCyan": "#56B6C2",
            "brightGreen": "#98C379",
            "brightPurple": "#C678DD",
            "brightRed": "#E06C75",
            "brightWhite": "#DCDFE4",
            "brightYellow": "#E5C07B",
            "cursorColor": "#FFFFFF",
            "cyan": "#56B6C2",
            "foreground": "#DCDFE4",
            "green": "#98C379",
            "name": "One Half Dark",
            "purple": "#C678DD",
            "red": "#E06C75",
            "selectionBackground": "#FFFFFF",
            "white": "#DCDFE4",
            "yellow": "#E5C07B"
        },
        {
            "background": "#FAFAFA",
            "black": "#383A42",
            "blue": "#0184BC",
            "brightBlack": "#4F525D",
            "brightBlue": "#61AFEF",
            "brightCyan": "#56B5C1",
            "brightGreen": "#98C379",
            "brightPurple": "#C577DD",
            "brightRed": "#DF6C75",
            "brightWhite": "#FFFFFF",
            "brightYellow": "#E4C07A",
            "cursorColor": "#4F525D",
            "cyan": "#0997B3",
            "foreground": "#383A42",
            "green": "#50A14F",
            "name": "One Half Light",
            "purple": "#A626A4",
            "red": "#E45649",
            "selectionBackground": "#FFFFFF",
            "white": "#FAFAFA",
            "yellow": "#C18301"
        },
        {
            "background": "#002B36",
            "black": "#002B36",
            "blue": "#268BD2",
            "brightBlack": "#073642",
            "brightBlue": "#839496",
            "brightCyan": "#93A1A1",
            "brightGreen": "#586E75",
            "brightPurple": "#6C71C4",
            "brightRed": "#CB4B16",
            "brightWhite": "#FDF6E3",
            "brightYellow": "#657B83",
            "cursorColor": "#FFFFFF",
            "cyan": "#2AA198",
            "foreground": "#839496",
            "green": "#859900",
            "name": "Solarized Dark",
            "purple": "#D33682",
            "red": "#DC322F",
            "selectionBackground": "#FFFFFF",
            "white": "#EEE8D5",
            "yellow": "#B58900"
        },
        {
            "background": "#FDF6E3",
            "black": "#002B36",
            "blue": "#268BD2",
            "brightBlack": "#073642",
            "brightBlue": "#839496",
            "brightCyan": "#93A1A1",
            "brightGreen": "#586E75",
            "brightPurple": "#6C71C4",
            "brightRed": "#CB4B16",
            "brightWhite": "#FDF6E3",
            "brightYellow": "#657B83",
            "cursorColor": "#002B36",
            "cyan": "#2AA198",
            "foreground": "#657B83",
            "green": "#859900",
            "name": "Solarized Light",
            "purple": "#D33682",
            "red": "#DC322F",
            "selectionBackground": "#FFFFFF",
            "white": "#EEE8D5",
            "yellow": "#B58900"
        },
        {
            "background": "#000000",
            "black": "#000000",
            "blue": "#3465A4",
            "brightBlack": "#555753",
            "brightBlue": "#729FCF",
            "brightCyan": "#34E2E2",
            "brightGreen": "#8AE234",
            "brightPurple": "#AD7FA8",
            "brightRed": "#EF2929",
            "brightWhite": "#EEEEEC",
            "brightYellow": "#FCE94F",
            "cursorColor": "#FFFFFF",
            "cyan": "#06989A",
            "foreground": "#D3D7CF",
            "green": "#4E9A06",
            "name": "Tango Dark",
            "purple": "#75507B",
            "red": "#CC0000",
            "selectionBackground": "#FFFFFF",
            "white": "#D3D7CF",
            "yellow": "#C4A000"
        },
        {
            "background": "#FFFFFF",
            "black": "#000000",
            "blue": "#3465A4",
            "brightBlack": "#555753",
            "brightBlue": "#729FCF",
            "brightCyan": "#34E2E2",
            "brightGreen": "#8AE234",
            "brightPurple": "#AD7FA8",
            "brightRed": "#EF2929",
            "brightWhite": "#EEEEEC",
            "brightYellow": "#FCE94F",
            "cursorColor": "#000000",
            "cyan": "#06989A",
            "foreground": "#555753",
            "green": "#4E9A06",
            "name": "Tango Light",
            "purple": "#75507B",
            "red": "#CC0000",
            "selectionBackground": "#FFFFFF",
            "white": "#D3D7CF",
            "yellow": "#C4A000"
        },
        {
            "background": "#000000",
            "black": "#000000",
            "blue": "#000080",
            "brightBlack": "#808080",
            "brightBlue": "#0000FF",
            "brightCyan": "#00FFFF",
            "brightGreen": "#00FF00",
            "brightPurple": "#FF00FF",
            "brightRed": "#FF0000",
            "brightWhite": "#FFFFFF",
            "brightYellow": "#FFFF00",
            "cursorColor": "#FFFFFF",
            "cyan": "#008080",
            "foreground": "#C0C0C0",
            "green": "#008000",
            "name": "Vintage",
            "purple": "#800080",
            "red": "#800000",
            "selectionBackground": "#FFFFFF",
            "white": "#C0C0C0",
            "yellow": "#808000"
        }
    ],
    "windowingBehavior": "useExisting"
}```