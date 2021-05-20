## usage

### shortcuts

- [windows](./windows.md)
- [windows terminal](./windows_terminal.md)
- [terminals on windows](./terminals.md)
- [readline](../tools/readline.md)
- [vscode](../toocls/vscode.md)
- [neovim](../tools/neovim.md)

### how to install windows software

- the install method doesn't matter that much, because now you can easily shim it to the intended environments
    - prefer the fewest installs that cover cmd/ps/mingw
    - prefer installing zips to C:\portable dir if app doesn't need any system integration, add shims using shimzon if needed
- powershell modules
    - install through Install-Module
    - unavailable in other shells, but that's probably just fine
- msix apps
    - just install from the store
    - apps available in command line through C:\Users\qasto\AppData\Local\Microsoft\WindowsApps path entry (execution aliases)
- mingw64 native apps
    - install through pacman
    - if needs to be available in msys you can shim it there, or install the msys-equivalent module
    - env_mingw to bring to PATH, use shimzon to add to other envs
- msys posix apps
    - install through pacman
    - available in mingw64 shells through fallback
    - env_msys to bring to PATH, use shimzon to add to other envs

## setup steps

### enable windows developer config

windows search -> developer settings -> developer mode -> On

### enable utf8 in the commandline (and everywhere else)

windows search -> advanced language settings -> Use Unicode-UTF-8 for worldwide language support

### enable multi-clipboard

- windows search -> clipboard settings -> clipboard history
- win+v allows pasting previous entries

### enable long paths for long-path-aware apps

- set `Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem\LongPathsEnabled` to 1 in regedit
- [docs](https://docs.microsoft.com/en-us/windows/win32/fileio/maximum-file-path-limitation)

### install redistributables

- <https://support.microsoft.com/en-us/topic/the-latest-supported-visual-c-downloads-2647da03-1eea-4433-9aff-95f26a218cc0>

### install fonts

* Install https://github.com/adam7/delugia-code/releases font, should be in portable/DelugiaCode
* right click and select install for all users, on the Complete variants of the fonts
* apply the registry key to configure console fonts
* add `"fontFace": "Delugia Mono Nerd Font",` to the default section of the windows terminal settings
* also install latest version of cascadia code ttf files, should be in portable/cascadiacode

### replace altgr key with ralt

* the only way to have windows recognize right alt as right alt(instead of remapping it to left-alt or other key) without it being altgr is to remove the altgr combinations from the layout
* install the layout from portable\uknoaltg
    * run setup exe
    * start -> language settings -> english -> options -> remove the default layout and select the new one
    * switch keyboard layout to the new custom one added by the installer in the tray
    * disable switching layouts using ctrl+shift in the language bar options
    * now ralt is just ralt, not altgr/lalt! and can be bound separately using things like autohotkey and some app shortcuts
* this layout can be recreated by using microsoft keyboard layout creator
    * install the software
    * load the uk keyboard layout
    * delete the keys for altgr and shift+altgr

### disable gefore experience (because it sets up global shortcuts)

- find NVidia LocalDisplay Container ("Container service for NVIDIA root features") -> properties -> startup type -> disabled

### install powershell 7

* `choco install powershell-core --install-arguments='"ADD_FILE_CONTEXT_MENU_RUNPOWERSHELL=0"'`
* in powershell 7:
    * `Update-Help`
    * hardlink profile files (sourcing seems to not work well, psreadline breaks for some reason)
```
fsutil hardlink create $profile.CurrentUserAllHosts $env:USERPROFILE\Documents\WindowsPowerShell\profile.ps1
fsutil hardlink create $profile.CurrentUserCurrentHost $env:USERPROFILE\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1
```
* $env:PSModulePath will contain both old and new module directories

### organize path variables

1. Add path variables for using in different definitons (contents themselves must not have variables, otherwise substitution won't work)
- PATH_VAR_NATIVE - windows shells + mingw
    - chocolatey
    - `C:\portable\SysinternalsSuite`
    - gitextensions
    - `C:\portable\bin\native`
- PATH_VAR_WINDOWS - all shells on windows including msys
    - powershell7
    - `C:\portable\msys\cmd`
    - cargo, dotnet, etc
    - most of what installers install
    - `C:\portable\bin\windows`
- PATH_VAR_ALL - all shells including wsl
    - vscode 
    - `C:\portable\bin\all`
2. configure environment level env variables to handle different environments
- PATH_APPEND_MINGW
    - %PATH_VAR_NATIVE%
- PATH_APPEND_BASH_COMMON
    - `C:\portable\bin\msix_apps`
    - `C:\portable\bin\bash`
- PATH_APPEND_MSYS_COMMON - software to append to path for all msys envs (msys, mingw)
    - `C:\portable\bin\msys_common`
    - %PATH_VAR_WINDOWS%
    - %PATH_VAR_ALL%
- PATH - software for windows shells
    - `C:\portable\bin\cmd`
    - `C:\portable\bin\winshell`
    - winapps
    - %PATH_VAR_NATIVE%
    - %PATH_VAR_WINDOWS%
    - %PATH_VAR_ALL%
- available PATH categories listed here for reference:
    - PATH_APPEND_MSYS_COMMON - msys+mingw
    - PATH_APPEND_MSYS - 
    - PATH_APPEND_MINGW - 
    - PATH_APPEND_WSL ; unimplemented
    - PATH_APPEND_POSIX_COMMON - msys+wsl
    - PATH_APPEND_BASH_COMMON - all bash shells 
- the variable loading should be already implemented in the shell profile files
2. example script
```
todo: this is out of date
New-ItemProperty -Name PATH_VAR_ALL -PropertyType String -Value "C:\Users\qasto\AppData\Local\Programs\Microsoft VS Code\bin;" -Path HKCU:\Environment
New-ItemProperty -Name PATH_VAR_WINDOWS -PropertyType String -Value "C:\Users\qasto\.cargo\bin;C:\Program Files\PowerShell\7\;C:\Program Files\dotnet\;C:\portable\msys\cmd;" -Path HKCU:\Environment
New-ItemProperty -Name PATH_VAR_WINDOWS -PropertyType String -Value "C:\ProgramData\chocolatey\bin;C:\portable\SysinternalsSuite;C:\Program Files (x86)\GitExtensions\;" -Path HKCU:\Environment

New-ItemProperty -Name PATH_APPEND_MINGW -PropertyType String -Value "%PATH_VAR_NATIVE%;" -Path HKCU:\Environment
New-ItemProperty -Name PATH_APPEND_MSYS_COMMON -PropertyType String -Value "C:\portable\portable_env\bash;%PATH_VAR_WINDOWS%;%PATH_VAR_ALL%" -Path HKCU:\Environment
$prevpath = (get-itemproperty -Path HKCU:\Environment).Path
Set-ItemProperty -Name PATH_APPEND_MSYS_COMMON -PropertyType String -Value "$prevpath;C:\portable\portable_env\cmd;%PATH_VAR_NATIVE%;%PATH_VAR_WINDOWS%;%PATH_VAR_ALL%" -Path HKCU:\Environment
```

### setup `portable\bin`

1. in `c:\portable\bin` make the following directory structure to store scripts, standalone binaries and shims
- all - all environments
- windows - native + msys
- winshell - cmd + powershell + explorer
- native - winshell + mingw
- cmd/ps/bash - shell-specific shims
- msix_apps - aliases for the winapps dir
- msys_common - msys and mingw
2. install and run portable env
    * install https://github.com/QAston/portable_env to c:\portable\bin
    * run `C:\portable\bin\portable_env` to regenerate `env_` scripts 
    * add the following to `%USERPROFILE%\Documents\WindowsPowerShell\profile.ps1`
```
$env:PATH="C:\portable\bin\ps;$env:PATH"
$env:PATHEXT=".PS1;$env:PATHEXT"
```
3. install [shimzon](git@github.com:QAston/shimzon.git) to `C:\portable\shimzon\bin\` to be able to add shims to various environments
    * add a shim to shimzon itself c:\portable\shimzon\bin\shimzon.exe add -d all c:\portable\shimzon\bin\shimzon.exe 

### set up msys2

1. follow [msys2 setup guide](../msys2.md)
2. set up shims for integrating with msys2
```
shimzon add  -d winshell -n bash-mingw64.exe C:\portable\msys\usr\bin\bash.exe
# this one has to be edited to add envvars:
[env]

MSYSTEM = "MINGW64"
CHERE_INVOKING = "true"
MSYS2_PATH_TYPE = "minimal"

shimzon add  -d winshell -n bash-msys.exe C:\portable\msys\usr\bin\bash.exe
# this one has to be edited to add envvars:
[env]

MSYSTEM = "MSYS"
CHERE_INVOKING = "true"
MSYS2_PATH_TYPE = "minimal"
```

### install symlink utils

1. in admin shell
```
# adds right-click-drag menu as well as Pin Link Source and Drop Link menus in RMB
choco install linkshellextension
```
2. download portable tools and add them to `all` shim
- add http://www.schinagl.priv.at/nt/xdel/xdel64.zip
- add https://schinagl.priv.at/nt/ln/ln64static.zip (rename to xln)
- add http://schinagl.priv.at/nt/dupemerge/dupemerge64.zip

### install msix packaging tool

store -> msix packaging tool

### install powertoys

* enable start at startup
* enable always run as admin
* disable everything except fancy zones and run
* configure run
    * change shortcut to win-x
    * change num results to 8
    * [usage](https://docs.microsoft.com/en-gb/windows/powertoys/run#action-key)
        * search prefixes:
            * `?` - file search
            * `//` - open browser
            * `<` - switch to window by name/title
            * `>` - run shell command
            * `:` - search registry key
            * `!` - search windows service
        * commands
            * Shutdown/Restart/Sign Out/Lock/Sleep/Hibernate
        * executables
            * you can add command line args
* configure fancy zones
    * make a vertical zone split in 2 for vertical monitor
    * enable hold shift to drag window to a zone
        * also, when holding control while moving between zones the window will be assigned to multiple zones
    * enable overriding windows snap keys and moving between 2 monitors
    * in the zone editors disable borders
    * you can do win+' to switch the layouts
    * mouse movement for windows snap still works

### install windows terminal

* install from github
* disable builtin shell menu: `REG ADD "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Shell Extensions\Blocked" /v "{9F156763-7844-4DC4-B2B1-901F640F5155}" /d "WindowsTerminal"`
* install https://github.com/lextm/windowsterminal-shell cloned into C:/portable
    - run .\install.ps1
    - or .\install.ps1 -PreRelease for terminal preview
* configure windows terminal to start in an existing cmd
```
"multiLinePasteWarning": false,
"windowingBehavior": "useExisting"
```
```
        "defaults":
        {
            // Put settings here that you want to apply to all profiles.
            "fontFace": "Delugia Mono Nerd Font",
            "altGrAliasing": false,
        },
```
* configure keybindings
```
        // scrollback config - same as my readline and vscode configs
        { "command": "scrollDown", "keys": "ctrl+down" },
        { "command": "scrollDownPage", "keys": "ctrl+pgdn" },
        { "command": "scrollUp", "keys": "ctrl+up" },
        { "command": "scrollUpPage", "keys": "ctrl+pgup" },
        { "command": "scrollToTop", "keys": "ctrl+home" },
        { "command": "scrollToBottom", "keys": "ctrl+end" },
```
* sendto shim-msix_apps C:\Users\qasto\AppData\Local\Microsoft\WindowsApps\wt.exe

### configure cmd.exe
1. better theme using concfg
```
C:\portable\concfg\concfg.ps1 import C:\portable\concfg\presets\qasto.json
```
2. readline for bat, download clink setup from https://chrisant996.github.io/clink/
```
# cmd.exe has autorun configured in registry, clink edits that configuration to add running it's own command
clink autorun install
```
3. completion
```
# git clone https://github.com/vladimir-kotikov/clink-completions to c:/portable/clink-completions if not already there
clink installscripts "C:\portable\clink-completions "
# git clone git@github.com:fredjoseph/clink-completions.git additional-git-completions
clink installscripts "C:\portable\additional-clink-completions "
```
4. set up prompt
```
clink installscripts "C:\portable\clink-prompt "
```
5. check clink info to see if scripts paths are picked up, use clink-prompt/clink-install.reg to fix if needed
6. set up config
```
# in shell:
clink set clink.paste_crlf  crlf
clink set history.max_lines 10000
#clink config: https://chrisant996.github.io/clink/ https://chrisant996.github.io/clink/clink.html and "old console config and shortcuts"
```

```
# in .inputrc
# add inside emacs block
  $if clink
     # intercepted and handled by conhost terminal   clink-scroll-line-up # ctrl up - scroll up
     # intercepted and handled by conhost terminal   clink-scroll-line-down# ctrl down - scroll down
    "\e[5;5~": clink-scroll-page-up # ctrl pgup - scroll page up
    "\e[6;5~": clink-scroll-page-down # ctrl pgdown - scroll page down
    "\e[1;5H": clink-scroll-top # ctrl home - scroll beginning
    "\e[1;5F": clink-scroll-bottom # ctrl end - scroll end
    "\C-z": nop # unbind undo
  $endif
```

### configure windows powershell 5.1

1. upgrade psget, run from admin powershell
```
Install-Module -Name PowerShellGet -Force
```
2. upgrade PSReadline, run from cmd:
```
powershell -noprofile -command "Install-Module PSReadLine -Force -SkipPublisherCheck -AllowPrerelease"
```
configure PSReadLine based on <https://github.com/PowerShell/PSReadLine/blob/master/PSReadLine/SamplePSReadLineProfile.ps1>, add to Microsoft.PowerShell_profile.ps1
```
using namespace System.Management.Automation
using namespace System.Management.Automation.Language

Import-Module -Name posh-git
Import-Module PSReadLine

Set-PSReadLineOption -EditMode Emacs
#Set-PSReadLineOption -HistorySearchCursorMovesToEnd
Set-PSReadLineKeyHandler -Key Ctrl+p -Function HistorySearchBackward
Set-PSReadLineKeyHandler -Key Ctrl+n -Function HistorySearchForward

# Clipboard interaction is bound by default in Windows mode, but not Emacs mode.
Set-PSReadLineKeyHandler -Key Ctrl+C -Function Copy
Set-PSReadLineKeyHandler -Key Ctrl+v -Function Paste
Set-PSReadLineKeyHandler -Key Ctrl+V -Function Paste

# Familiar windows keybingdings for navigation and deletion
Set-PSReadLineKeyHandler -Key Ctrl+Delete -Function ShellKillWord
Set-PSReadLineKeyHandler -Key Ctrl+Backspace -Function ShellBackwardKillWord
Set-PSReadLineKeyHandler -Key Ctrl+LeftArrow -Function ShellBackwardWord
Set-PSReadLineKeyHandler -Key Ctrl+RightArrow -Function ShellForwardWord
Set-PSReadLineKeyHandler -Key Ctrl+Shift+LeftArrow -Function SelectShellBackwardWord
Set-PSReadLineKeyHandler -Key Ctrl+Shift+RightArrow -Function SelectShellForwardWord

# common keybinds for scrolling
Set-PSReadLineKeyHandler -Key Ctrl+PageDown -Function ScrollDisplayDown
Set-PSReadLineKeyHandler -Key Ctrl+PageUp -Function ScrollDisplayUp
Set-PSReadLineKeyHandler -Key Ctrl+DownArrow -Function ScrollDisplayDownLine
Set-PSReadLineKeyHandler -Key Ctrl+UpArrow -Function ScrollDisplayUpLine
Set-PSReadLineKeyHandler -Key Ctrl+End -Function ScrollDisplayToCursor
Set-PSReadLineKeyHandler -Key Ctrl+Home -Function ScrollDisplayTop

# display predictions
Set-PSReadLineOption -PredictionSource History

# F1 for help on the command line - naturally
Set-PSReadLineKeyHandler -Key F1 `
                         -BriefDescription CommandHelp `
                         -LongDescription "Open the help window for the current command" `
                         -ScriptBlock {
    param($key, $arg)

    $ast = $null
    $tokens = $null
    $errors = $null
    $cursor = $null
    [Microsoft.PowerShell.PSConsoleReadLine]::GetBufferState([ref]$ast, [ref]$tokens, [ref]$errors, [ref]$cursor)

    $commandAst = $ast.FindAll( {
        $node = $args[0]
        $node -is [CommandAst] -and
            $node.Extent.StartOffset -le $cursor -and
            $node.Extent.EndOffset -ge $cursor
        }, $true) | Select-Object -Last 1

    if ($commandAst -ne $null)
    {
        $commandName = $commandAst.GetCommandName()
        if ($commandName -ne $null)
        {
            $command = $ExecutionContext.InvokeCommand.GetCommand($commandName, 'All')
            if ($command -is [AliasInfo])
            {
                $commandName = $command.ResolvedCommandName
            }

            if ($commandName -ne $null)
            {
                Get-Help $commandName -ShowWindow
            }
        }
    }
}
Set-PSReadLineOption -BellStyle None
Set-PSReadLineOption -MaximumHistoryCount 40960

Set-PoshPrompt -Theme "C:\portable\oh-my-posh\custom.omp.json"
```
4. add autocompletion plugins for installed commands: <https://www.powershellgallery.com/packages?q=Tags%3A%22tab-completion%22>
```
# install
Install-Module -Name posh-git
```
add to profile
```
Import-Module -Name posh-git
```
5. configure the prompt
```
Install-Module -Name oh-my-posh
```
add to profile
```
Set-PoshPrompt -Theme c:\portable\oh-my-posh\custom.omp.json
```
6. set SHELL variable for use in nvim and others
```
$env:SHELL=(Get-Process -Id $PID).Path
$env:SHELL_LANG=(Get-Process -Id $PID).ProcessName
```

### configure vscode

- see [vscode](../tools/vscode.md)

### set up send-to targets

1. Send to sets workdir to current dir and sets one parameter is set with the path of the file sent, which the software/script needs to handle
2. Go to shell:sendto in explorer
3. Delete entries you aren't going to use
4. Add entries for shells (portable/sendto-template exec-* files)
5. Add entries for adding shims (portable/sendto-template shim-* files)
6. Add entry for adding entries to start menu search: shell-alias.bat

### set up visual studio

- see [visual studio](./visual_studio.md)

### set up windows sdk

- see [windows sdk](./windows_sdk.md)

### set up sysinternals

- see [sysinternals](./sysinternals.md)

### set up wsl2 and docker

- see [wsl2_artix](./wsl2_artix.md)

### set up "everything search"

- install with installer, enable indexing service
- in preferences disable indexes for games and elements
- pin to windows bar
- run `Install-Module -Name PSEverything` in windows powershell
    - run `Search-Everything -Extension cpp,h  | Get-Item` to search

### set up rust 

follow [rust_tools](../rust/tools.md)

### set up node/npm

follow [npm](../tools/npm.md)

### set up ripgrep

- [download](https://github.com/BurntSushi/ripgrep/releases) msvc x64 zip and put in portable
- rg sendto shim-windows
- add completions
    - bash: add `. "C:\portable\ripgrep\complete\rg.bash"` to bashrc after /etc/bash_completion
    - powershell: add `. C:\portable\ripgrep\complete\_rg.ps1` to profile

### set up fd

- [download](https://github.com/sharkdp/fd/releases/) msvc x64 zip and put in portable
    - also download unknown-linux-gnu package and get the autocomplete folder and put it in portable
- fd sendto shim-windows
    - bash: `. "C:\portable\fd\autocomplete\fd.bash-completion"` to bashrc
    - powershell: add `. "C:\portable\fd\_fd.ps1"`

### add small unix tools to windows path

1. configure user env variables required by some unix tools
    - HOME="C:\Users\qasto\"
    - EDITOR="nvim"
2. from c:\portable\msys\usr\bin - only add utilities that actually work outside of msys
    - sendto rlwrap winshell
    - sendto wget winshell
    - sendto which winshell
    - sendto cygpath winshell
3. download some binaries ported to windows, so that they work outside of msys (for example clone git@github.com:bmatzelle/gow.git)
    - utilities which are shipped with msys, but don't work well outside of bash
    - sendto cat winshell
    - sendto xargs winshell
    - sendto sed winshell
4. download [jq](https://github.com/stedolan/jq) and add it to portable/bin/windows

### TODO: set up cross-shell aliases 

- doskey for cmd
- alias for bash
- New-Alias for powershell
- set up a shared file that is read on startup of each shell


### set up fzf

follow [fzf](../tools/fzf.md)

### set up nvim

follow [neovim](../tools/neovim.md)

### set up openjdk

- unpack the zip downloaded from <https://jdk.java.net/java-se-ri/16> to portable
- add an entry to portable_env:
```
jdk16 = [
    {command = 'env', key = 'JAVA_HOME', value = 'C:\portable\openjdk16\jdk-16\', mode = 'PATH'},
    {command = 'env', key = 'JRE_HOME', value = 'C:\portable\openjdk16\jdk-16\', mode = 'PATH'},
    {command = 'env', key = 'PATH', value = '%JAVA_HOME%\bin', mode = 'PREPEND_PATH'}
]
```

### set up ghidra

- download <https://ghidra-sre.org/> to portable
- edit runGhidra.bat to add env_jdk11 call so java is found (must be jdk11, otherwise it'll complain)

## future todos

- c++ ide?
- python setup with readline? 
