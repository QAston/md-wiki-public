## requirements

- usable linux distro - done by wsl2, see [windows/wsl2_artix]()
- windows
    - compiler toolchains
    - convenient shell
    - system package management
    - powertoys?

## setup steps

### 1. copy portable dir and update system config
1. copy c:/portable dir
2. install https://github.com/QAston/PEIS, which should be in portable/Utils
* add the following to `%USERPROFILE%\Documents\WindowsPowerShell\profile.ps1`
```
$env:PATH="C:\portable\utils\ps;$env:PATH"
$env:PATHEXT=".PS1;$env:PATHEXT"
```
* add `C:\portable\utils` and `C:\portable\utils\cmd` to the user PATH variable
* add to bash envs?

### 2. install [scoop](./scoop.md)

### 3. configure cmd.exe
1. better theme
```
scoop install concfg
concfg import default-dark
```
2. readline for bat
```
scoop install clink
# cmd.exe has autorun configured in registry, clink edits that configuration to add running it's own command
clink autorun install
```
3. completion
```
# git clone https://github.com/vladimir-kotikov/clink-completions to c:/portable/clink-completions if not already there
clink installscripts C:\portable\clink-completions
```
- todo: clink config: https://chrisant996.github.io/clink/ https://chrisant996.github.io/clink/clink.html
- todo: prompt? <https://github.com/chrisant996/cmder-powerline-prompt>

### 4. configure windows powershell 5.1

1. upgrade psget, run from admin powershell
```
Install-Module -Name PowerShellGet -Force
```
2. upgrade PSReadline, run from cmd:
```
powershell -noprofile -command "Install-Module PSReadLine -Force -SkipPublisherCheck -AllowPrerelease"
```
configure PSReadLine based on <https://github.com/PowerShell/PSReadLine/blob/master/PSReadLine/SamplePSReadLineProfile.ps1>, add to profile.ps1
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

# todo: once more predictors are available for new powershell 7.1+ predictor api use psreadline 2.2.0+ and set
# Set-PSReadLineOption -PredictionSource HistoryAndPlugin # will not work in old powershell
```
4. install tools for better powershell prompt
```
scoop install pshazz
```
make sure to add 
```
Set-Alias -Name pshazz -Value "C:\Users\qasto\scoop\apps\pshazz\current\bin\pshazz.ps1"
```
to the `%USERPROFILE%\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1` file, otherwise pshazz won't be found because scoop isn't in the path by default

TODO: configure powershell 7

### 5. todo
- todo:  https://github.com/git-for-windows/git/wiki/Install-or-update-inside-MSYS2,-Cygwin-or-Git-for-windows-itself