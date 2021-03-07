## usage

- wsl2 - use as a linux machine that can access windows drive
- windows - multiple environments that ca

### how to install windows software

- the install method doesn't matter that much, because now you can easily shim it to the intended environments
    - prefer the fewest installs that cover cmd/ps/mingw
    - don't add software to incompatible envs
- powershell modules
    - install through Install-Module
    - unavailable in other shells, but that's probably just fine
- msix apps
    - just install from the store
    - apps available in command line through C:\Users\qasto\AppData\Local\Microsoft\WindowsApps path entry (execution aliases)
    - add a shim using shimzon
- mingw64 native apps
    - install through pacman
    - available in other command lines through env_mingw? (TODO: fix env_mingw)
    - if needs to be available in msys you can shim it there, or install the msys-equivalent module
- msys posix apps
    - install through pacman
    - available in mingw64 shells through fallback
    - available in other command lines through env_msys (TODO)
    - if needs to be available in other shell make a shim for other shells? (TODO)
- C:/portable apps
    - if needs to be available in shells add a shim to bin to the appropriate dir
- msi apps
- chocolatey apps

## setup steps

### enable windows developer config

windows search -> developer settings -> developer mode -> On

### enable utf8 in the commandline (and everywhere else)

windows search -> advanced language settings -> Use Unicode-UTF-8 for worldwide language support

### enable multi-clipboard

- windows search -> clipboard settings -> clipboard history
- win+v allows pasting previous entries

### enable long paths for long-path-aware apps

- set Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem\LongPathsEnabled to 1 in regedit
- [docs](https://docs.microsoft.com/en-us/windows/win32/fileio/maximum-file-path-limitation)

### install redistributables

- <https://support.microsoft.com/en-us/topic/the-latest-supported-visual-c-downloads-2647da03-1eea-4433-9aff-95f26a218cc0>

### copy portable dir and update system config
1. copy c:/portable dir
2. install https://github.com/QAston/PEIS, which should be in portable/Utils
* add the following to `%USERPROFILE%\Documents\WindowsPowerShell\profile.ps1`
```
$env:PATH="C:\portable\bin\ps;$env:PATH"
$env:PATHEXT=".PS1;$env:PATHEXT"
```

### install fonts

* Install https://github.com/adam7/delugia-code/releases font, should be in portable/DelugiaCode
* right click and select install for all users, on the Complete variants of the fonts
* apply the registry key to configure console fonts
* add `"fontFace": "Delugia Mono Nerd Font",` to the default section of the windows terminal settings
* also install latest version of cascadia code ttf files, should be in portable/cascadiacode

### install powershell 7

* `choco install powershell-core --install-arguments='"ADD_FILE_CONTEXT_MENU_RUNPOWERSHELL=0"'`
* in powershell 7:
    * `Update-Help`
    * sorce windows powershell profile files
* hardlink profile files (sourcing seems to not work well)
```
New-Item -Force -Path $profile.CurrentUserAllHosts
New-Item -Force -Path $profile.CurrentUserCurrentHost
". $env:USERPROFILE\Documents\WindowsPowerShell\profile.ps1 " |  Out-File -FilePath $profile.CurrentUserAllHosts
". $env:USERPROFILE\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1 " |  Out-File -FilePath $profile.CurrentUserCurrentHost
```

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

### setup bin

1. c:\portable\bin has a following structure to store scripts, standalone binaries and shims
- all - all environments
- windows - native + msys
- winshell - cmd + powershell + explorer
- native - winshell + mingw
- cmd/ps/bash - shell-specific shims
- msix_apps - aliases for the winapps dir
2. C:\portable\shimzon\bin\shimzon has a tool to add shims to various envrionments, to add a shim run:
```
shimzon add path\to\exe -d <envname>
```
3. shims to set up
```
c:\portable\shimzon\bin\shimzon.exe add -d all c:\portable\shimzon\bin\shimzon.exe 
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

shimzon add -d msix_apps C:\Users\qasto\AppData\Local\Microsoft\WindowsApps\wt.exe
```
4. run C:\portable\bin\portable_env to regenerate `env_` scripts 


### install msix packaging tool

store -> msix packaging tool

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
# clink set clink.paste_crlf  todo: disable this somehow, luckily it's only broken outside wt
clink set history.max_lines 10000
#clink config: https://chrisant996.github.io/clink/ https://chrisant996.github.io/clink/clink.html and "old console config and shortcuts"
```

```
# in .inputrc
# add inside emacs block
  $if clink
    "\x1b[27;5;32~":    clink-popup-complete # ctrl+space
  $endif
  # todo: make bindings more in sync with standard bash keybindings here?
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
Import-Module PSReadLine
Set-PSReadLineOption -EditMode Emacs
#Set-PSReadLineOption -HistorySearchCursorMovesToEnd
Set-PSReadLineKeyHandler -Key UpArrow -Function HistorySearchBackward
Set-PSReadLineKeyHandler -Key DownArrow -Function HistorySearchForward
# In Emacs mode - Tab acts like in bash, but the Windows style completion
# is still useful sometimes, so bind some keys so we can do both
Set-PSReadLineKeyHandler -Key Ctrl+q -Function TabCompleteNext
Set-PSReadLineKeyHandler -Key Ctrl+Q -Function TabCompletePrevious

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

Set-PSReadLineOption -PredictionSource History
# Set-PSReadLineOption -PredictionSource HistoryAndPlugin # will not work in old powershell
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

### configure terminals in vscode

1. install shell launcher plugin
2. Edit settings.json and add:
```
    #configure internal terminal (external terminal will open a separate window)
    "terminal.integrated.fontFamily": "'Delugia Mono Nerd Font'",
    "terminal.integrated.shell.windows": "C:\\Windows\\System32\\cmd.exe",
    "shellLauncher.shells.windows": [
        {
            "shell": "C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe",
            "label": "PowerShell"
        },
        {
            "shell": "C:\\Windows\\System32\\cmd.exe",
            "label": "cmd"
        },
        {
            "shell": "C:\\Windows\\System32\\wsl.exe",
            "label": "Artix wsl"
        },
        {
            "shell": "C:\\Program Files\\PowerShell\\7\\pwsh.exe",
            "label": "pwsh"
        },
        {
            "shell": "C:\\portable\\msys\\msys2_shell.cmd",
            "args": ["-defterm", "-here", "-no-start", "-mingw64"],
            "label": "mingw64"
        },
        {
            "shell": "C:\\portable\\msys\\msys2_shell.cmd",
            "args": ["-defterm", "-here", "-no-start", "-msys"],
            "label": "msys"
        }
    ],
```
3. To launch the terminal do Ctrl+shift+P -> Shell launch -> choose shell

### set up send-to targets

1. Send to sets workdir to current dir and sets one parameter is set with the path of the file sent, which the software/script needs to handle
2. Go to shell:sendto in explorer
3. Delete entries you aren't going to use
4. Add entries for shells (portable/sendto-template exec-* files)
5. Add entries for adding shims (portable/sendto-template shim-* files)
6. Add entry for adding entries to start menu search: shell-alias.bat

### set up visual studio

- install visual studio
- add modules using visual studio installer
    - desktop c++ development
    - uwp development
- add components using the installer
    - clang compiler
    - clang-cl
- set up scripts for dev shell and add to portable\bin\cmd & ps
    - take contents from shortcuts to the developer prompt/powershell from start menu
    - env_vs.bat:
```
call "C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\Common7\Tools\VsDevCmd.bat"
```
    - env_vs.ps1
```
Import-Module "C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\Common7\Tools\Microsoft.VisualStudio.DevShell.dll"; Enter-VsDevShell e821c35b
```

### set up wsl

- see [windows/wsl2_artix]()

### set up "everything search"

- install with installer, enable indexing service
- in preferences disable indexes for games and elements
- pin to windows bar

### todos

- powertoys
- windows shortcuts
- [fzf](https://github.com/junegunn/fzf) 
    - fzf based tools https://github.com/tadashi-aikawa/owl-cmder-tools
- ripgrep
- [fd](https://github.com/sharkdp/fd)
- readline shortcuts  
- vscode shortcuts and plugins
- <https://github.com/anishathalye/dotbot>
- <https://github.com/imachug/win-sudo>