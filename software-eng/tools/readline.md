## overview

- <https://wiki.archlinux.org/index.php/readline>
- <https://catonmat.net/ftp/bash-vi-editing-mode-cheat-sheet.pdf>
- <https://catonmat.net/ftp/readline-emacs-editing-mode-cheat-sheet.pdf>
- some applications have option to have readline but don't ship it by default (like python on windows for example)


### arguments

- pressing alt-number (can be multiple digit) before a command passes a numeric argument to the command
- by default it's repetition
- argument will be visible in prompt
- number can be negative, some commands interpret it as reversing the direction where the command is applied

### keybinding notation

`\m-k` - means alt+k
alternatively you can press esc, then immediately k (on keyboards without alt only?)
`\c-x s` - means ctrl - x then s

### vi vs emacs mode

- readline uses emacs mode (and emacs-standard keymap) by default
- you can configure prompt based on the mode (off by default):
    - set show-mode-in-prompt on/off
    - set emacs-mode-string text
    - set vi-cmd-mode-string text
    - set vi-ins-mode-string text
- switching the mode in vi switches the keymaps between vi-command and vi-insert
- mode switching shortcuts won't work in bash, you need to use set -o vi instead

### initrc

- $term value is the value of the $TERM variable, on msys by default xterm-256colors, linux with no gui has "linux"
    - mintty will set this value to the value configured in settigs (you can choose which terminal it will try to emulate)

### history

- history is a set of lines populated based on the config options
    - depending on the settings it's not a straight replay of all commited lines, for example usually you don't want to store duplicates
    - up/down moves in the history, Alt-> to end Alt-< to beginning
- search
    - C-p/C-d moves to the previous/next line that starts with what's in the line before cursor
    - Alt-p/Alt-d moves to the previous/next line that contains the searched string
- some history options are options of the application rather than readline (like where history is stored, when it is appended, etc)
- history -a to flush history to other shells; history -r to reload; history -c to clear
- incremental search (C-s - forward, C-r - backward)
    - will stop on the first find forward/backward from the starting place in history (pressing incremental search again searches for next match)
    - doesn't wrap around, so if you add more text you might need to search in reverse direction, because the every search attempt moves you in the history
        - or go back to the end of history Alt+> to reset the search, sadly that doesn't work because moving history ends the search
    - the search term is displayed in the prompt
    - ESC and ctrl-J end the search
- configuration
    - consecutive duplicate entries are removed (in the usual config)

### history references

- you can refer to history cmdline entry using notations:
    - !n - history entry n
    - !-n - history n lines back
    - !! - same as !-1
    - !string - most recent command starting with string
    - !?string - most recent command preceding current history pos starting with string
- you can specify which part of entry to use by using `!!:<designator>`:
    - n - nth word (0 - command line)
    - $ - last word
    - ^ - 1st word
    - x-y - range of words
    - -y - 0-y
    - * - 1-$
    - x* - x-$
    - x- like x* but omits last word
- you can add a modifier after a designator `!!:n:<designator>` (can add multiple, each preceded with :):
    - s/old/new - replaces old with new in the line
    - p - print but don't execute
    - h - leave only head of path
    - t - remove head of path
- [details](https://tiswww.case.edu/php/chet/readline/history.html) 
    

