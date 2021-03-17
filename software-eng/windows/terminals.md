# windows terminal software (conhost and extensions)

- <https://mintty.github.io/> (see below)
- <https://github.com/cbucher/console>
- [conemu](https://conemu.github.io/) - a good terminal and a great repository of info on command line tooling
- [windows terminal](./windows_terminal.md) - a terminal replacement from ms
- <https://cmder.net/>
- <https://www.putty.org/>
- [terminals vs shell msys](https://www.msys2.org/wiki/Terminals/)
- <https://github.com/KUTlime/PowerShell-Open-Here-Module>


### builtin terminal (conhost)

* `HKEY_CURRENT_USER\Software\Microsoft\Command Processor\AutoRun` - `cmd.exe` autorun
    * "C:\Program Files (x86)\clink\1.1.34.d161e9\clink.bat" inject --autorun --profile ~\clink
* new conhost - handle of xterm/vt100 terminal sequences:
    * [handled terminal sequences](https://docs.microsoft.com/en-us/windows/console/console-virtual-terminal-sequences#input-sequences)
    * keycodes:
        * alt+shift+ctrl modifiers work on the f-keys, arrows, backspace, delete, pgup/down home/end (they yeld different sequences)
        * alt works for every base keypress because alt modifier is encoded (see [](../unix/terminals.md)) as `ESC nonaltchar`
        * ctlr characters are only supported in C0 ascii ctrl range (ctrl-space, ctrl-[, ctrl-], ctrl-^, ctrl-_) because they're encoded using C0 ASCII control codes. As a result ctrl+shift+char, ctrl+num, and in general ctrl with other ascii characters is ignored or reproduces a keycode already bound to control range
        * keycodes can be inspected using `clink echo` on in msys using `ctrl-v` (quote) then keystroke
* [improvements to the default windows terminal](https://docs.microsoft.com/en-us/previous-versions/orphan-topics/ws.11/mt427362(v=ws.11)?redirectedfrom=MSDN)
    - quick edit mode 
        - select and right click/enter to copy
        - select while holding alt to select a square
        - right click/enter to paste
    - insert mode - set to true means you shift the text when inserting instead of overtyping
    - load_console_IME - whether to load a dll for asian language support, ctrl+space enables it supposedly?
    - LineSelection - enables line selecting instead of block (old default was only block selection, with this enabled block selection is done holding ALT)
    - FilterOnPaste - filters pasted text for smart quotes and tabs
    - LineWrap - rewraps text on window resize
    - TerminalScrolling - "disable scroll forward"?
    - HistoryBufferSize - number of accessible history entries using history commands and arrows
    - NumberOfHistoryBuffers - number of processes which get their individual buffer, over this number the history buffers will be recycled
    - ScreenBufferSize - the size of the buffer for scrolling output
    - ExtendedEdit Keys - new selection shortcuts
        - shift + `<key>`
    - TrimLeadingZeros - if doubleclicking a number should select leading zeros
    - WordDelimiters - where doubleclicking/word selecting should stop
    - ForceV2 - enable new features
    - CtrlKeyShortcutsDisabled - enables/disables keybindings:
        - CRTL+V or SHIFT+INS - paste
        - CTRL+C - with selection first press copies (and clears text), second sends BREAK, without selection BREAK
        - CTRL+INS - copy
        - CTRL+HOME/END - move to beginning/end of the buffer if empty commandline, otherwise delete stuff to the left/right of the cursor
    - more settings can be found in Computer\HKEY_CURRENT_USER\Console
    - settings can be automated using C:/portable/concfg/lib/core.ps1 keys
    - <https://www.itprotoday.com/compute-engines/undocumented-command-prompt-tips>
* shortcuts
    - output(buffer) history
        - ctrl + up / ctrl + down - scroll the text up/down in the window
        - ctrl + home/end - scroll to beginning/end of buffer
        - ctrl + f - window "find in buffer", allows finding next/prev occurance of text
    - transparency
        - ctrl + `<plus>/<minus>/<scrollup>/<scollbottom>`
    - alt + enter/F11 - fullscreen
    - ctrl +c - interrupt (when nothing is selected)
    - ctrl + m - selection mode
        - allows selecting text output using only keyboard
        - navigation keys (arrow/page/home/end/ctrl-home/ctrl-end) - move cursor to target
        - shift while navigating select text from current to target position
        - `<esc>` exits
    - selecting in regular mode
        - shift + navigation keys lets you select from output buffer, starting from the current input line, each navigation command extends the selection
        - ctrl + a - Select all text after the prompt, if the cursor is in the current line and the line is not empty
        - ctrl + a - select everything in the buffer if the cursor is not in the current line
        - CTRL+C - copy selection and deselect
        - CRTL+V or SHIFT+INS - paste
* shell shortcuts - powershell amd cmd.exe only
    - command manipulation
        - ESC - clear the text
        - CTRL+home - delete all the characters to the left (if not empty)
        - CTRL+end - delete all characters to the right (if not empty)
        - CTRL+arrows - word movement
    - command history
        - F1 - bring 1 character from prev command
        - F2 `<char>` - bring characters from prev command up to character `<char>`
        - F3 - bring the entire prev command to current text
        - F4 `<char>` - delete from current text up to character `<char>`
        - F5 - move 1 back in history
        - F6 - ^Z character
        - F7 - history list menu
        - Alt-F7 - clear history
        - F8 - search backwards for command matching the current text
        - F9 - run a command by giving history line number
* clink - readline support for commandline shell executable
    - links
        * <https://chrisant996.github.io/clink/>
        * <https://chrisant996.github.io/clink/clink.html> 
        * <https://github.com/vladimir-kotikov/clink-completions>
        * <https://github.com/chrisant996/cmder-powerline-prompt>
        * <https://github.com/HelloWorld017/clink-powerline>
        * <https://github.com/skywind3000/z.lua>
    - clink disables ALL default keyboard input handling for the shell executable and replaces them with readline
        - other executables spawned from the shell are unaffected, as it only affects the shell process, not the terminal
        - you can still trigger mark mode by using the gui menu
        - used only for cmd, because powershell has it's own readline implementation
    - clink has it's own history mechanism so it ignores the conhost history buffer settings
    - `clink history` shows command history
    - `alt + h` shows bound commands
    - autocompletion
        - to skip executable matching do space, then tab completion
        - to skip hidden files in autocompletion set `files.hidden` to False
    - history
        - ! can be used to access history (as long as it's not in single/double quotes)
            - !-1 is previous command
        - set `history.max_lines` to the size of history stored
        - by default ignores lines starting with whitespace when adding to history, to prevent that set `history.ignore_space` to False
        - set `history.shared` to True if you want every command to merge to history immediately, otherwise they'll be merging on exit
        - set `history.dupe_mode` to change handling of duplicates (by default old entry is removed)
        - set `history.dont_add_to_history_cmds` to say which commands to ignore in history
        - history is saved to `c:\Users\<username>\AppData\Local\clink`
    - other settings
        - `clink.paste_crlf` is bad because it prevents multiple pastes, but it luckily doesn't apply in windows terminal
    - keybinds
        - most keybinds are the same as [readline](../tools/readline.md)
        - ESC - clear line text
    - `clink echo` prints most of the input escape codes handled by windows conhost
    - binding keyboard to a custom command: <https://github.com/genotrance/ff/issues/4>

### mintty

- the most xterm compatible terminal on windows (other than using xserver and wsl)
- [documentation](https://mintty.github.io/mintty.1.html)
- [wiki](https://github.com/mintty/mintty)
- example config:
```
Font=Delugia Mono Nerd Font
Term=xterm
BellType=0
TabBar=1
SessionGeomSync=3
AltGrIsAlsoAlt=yes
CtrlShiftShortcuts=yes
ConPTY=1
ScrollMod=4
```