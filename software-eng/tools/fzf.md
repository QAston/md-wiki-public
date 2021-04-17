## fzf

### setup windows

- [download](https://github.com/junegunn/fzf/releases) windows amd64 binary and put in portable
- clone git@github.com:junegunn/fzf.git into portable/fzf to get completion scripts
- fzf sendto shim-windows
- setup [powershell integration](https://github.com/kelleyma49/PSFzf)
    - run ` Install-Module -Name PSFzf ` in windows powerhsell
    - configure profile: 
```
Import-Module -Name PSFzf 
Set-PSReadLineKeyHandler -Key Shift+Tab -ScriptBlock { Invoke-FzfTabCompletion }
Set-PsFzfOption -PSReadlineChordReverseHistory 'Alt+s'
```
- setup [fuzzy selection for standard bash completion](https://github.com/lincheney/fzf-tab-completion) by adding to `.bashrc`
    - . "C:\portable\fzf\fzf-tab-completion\bash\fzf-bash-completion.sh"
    - bind -x '"\e[Z": fzf_bash_completion' # binds to shift+tab, don't bind to \t because the completion is too slow
- setup [history seach keybinding](https://github.com/junegunn/fzf/tree/master/shell)
    - comment out all keybindings that aren't ctrl+r
    - change ctrl+r to "\es"
    - add `. "C:\portable\fzf\fzf\shell\key-bindings.bash"` to bashrc
- setup clink/batch integration
    - make sure C:\portable\clink-prompt\fzf.lua is running
        - sets up shift-tab completion and shortcuts
    - add config to inputrc:
```
    "\es":	"luafunc:fzf_history" # alt+s
    "\e[Z":	"luafunc:fzf_completion" # ctrl+space
```
    - make sure xargs is in PATH
- replace default find commands with fd:
    - set windows env variable FZF_DEFAULT_COMMAND to `fd --type f --follow`
- todo: add some shortcut scripts for common fzf usage?

### setup wsl2

- pacman -S fzf
- copy the modified "C:\portable\fzf\fzf\shell\key-bindings.bash" file to ~/portable/fzf
```
cp "/mnt/c/portable/fzf/fzf/shell/key-bindings.bash" ~/portable/fzf/key-bindings.bash
```
- copy the modified "C:\portable\fzf\fzf-tab-completion\bash\fzf-bash-completion.sh" to ~/portable/fzf/fzf-bash-completion.sh
```
cp "/mnt/c/portable/fzf/fzf-tab-completion/bash/fzf-bash-completion.sh" ~/portable/fzf/
```
- add to ~/.bashrc:
```
source ~/portable/fzf/key-bindings.bash
source ~/portable/fzf/fzf-bash-completion.sh
bind -x '"\e[Z": fzf_bash_completion'
```

### usage

- a command which when piped let's you interactively choose which entry will be printed to stdout
    - `fzf -m` lets you output multiple entries, TAB to select SHIFT+TAB to deselect
    - enter to select
    - ESC, CTRL+G - abort search
    - CTRL+C - immediate abort search
- with no input fzf reads output of `$FZF_DEFAULT_COMMAND`
- shell keybindings
    - SHIFT+TAB - fuzzy tab completion
    - ALT+S - paste selected command from history
- passing output of fzf to application args
    - bash and powershell: `command $(fzf)`
    - all shells - use xargs: `fzf | xargs command`
    - powershell 
        - pipelines: `Get-ChildItem . -Recurse | ? { $_.PSIsContainer } | Invoke-Fzf | Set-Location`
        - Invoke-FuzzyGitStatus, Set-LocationFuzzyEverything, Invoke-FuzzySetLocation, Invoke-FuzzyHistory
- match syntax: 
    - sbtrkt - fuzzy-match Items that match sbtrkt
    - `'wild` - exact-match (quoted) Items that include wild
    - `^music`- prefix-exact-match Items that start with music
    - `.mp3$` - suffix-exact-match Items that end with .mp3
    - `!fire` - inverse-exact-match Items that do not include fire
    - `!^music` - inverse-prefix-exact-match Items that do not start with music
    - `!.mp3$` - inverse-suffix-exact-match Items that do not end with .mp3
    - `|` - logical or operator
- [fzf has key/event bindings](https://www.mankier.com/1/fzf#Key/Event_Bindings)

### fzf tools

- fzf also provides completion in places where you put `**` (`$FZF_COMPLETION_TRIGGER`) - <https://github.com/junegunn/fzf/tree/master/shell>
    - can replace the dir/path completions by overriding `_fzf_compgen_{path,dir}`
    - not really worth having
- fzf provides bash keybindings <https://github.com/junegunn/fzf/tree/master/shell>
    - ALT+C - select dir from result of `$FZF_ALT_C_COMMAND` - BROKEN
    - CTRL+T - use selected token as a root dir for search, select from result of `$FZF_CTRL_T_COMMAND` and paste
    - CTRL+R - paste selected command from history
    - key handling in bash keyboard shortcut impl (outside of these things are fine):
        - CTRL+G/ESC/Enter - all of these wait until the search command is finished, which isn't the case in cmd?
        - CTRL+C - immediate abort (command maybe still hanging in background?), breaks the entered readline command (either removes it completely or silently breaks the first character, also breaks the history for the command :()
        - set up fd search that's limited in depth/count for these commands?
- there's a good [powershell module](https://github.com/kelleyma49/PSFzf) providing shortcuts, history search and some additional commandlets 
- display standard bash completion using fzf <https://github.com/lincheney/fzf-tab-completion>: 
    - alternative impl: <https://github.com/rockandska/fzf-obc>
- <https://github.com/junegunn/fzf/wiki/Related-projects>
- convenience scripts:
    - <https://github.com/atweiden/fzf-extras>
    - <https://github.com/DanielFGray/fzf-scripts>
    - <https://github.com/3hhh/fzfuncs>
    - <https://github.com/junegunn/fzf/wiki/examples>
- there's plenty of git helpers
- <https://github.com/bigH/interactively>
- there's some projects which provide a reimplementation of xargs (bad, ignore them)
    - <https://github.com/genotrance/ff> - has shortcuts for searching for directories/files/extensions, but otherwise can be ignored as xargs is better
    - <https://github.com/jesse23/with> - really bad
- replacing default find commands with fd: <https://github.com/chhajedji/fzf/commit/002a3eb3a7282f32e071b7f42d7e3e1bebb36077>
