## overview

- <https://wiki.archlinux.org/index.php/readline>
- <https://catonmat.net/ftp/bash-vi-editing-mode-cheat-sheet.pdf>
- <https://catonmat.net/ftp/readline-emacs-editing-mode-cheat-sheet.pdf>
- some applications have option to have readline but don't ship it by default (like python on windows for example)

### readline wrappers

- you can make an application use readline by using one of the readline wrappers:
    - https://github.com/hanslub42/rlwrap
    - will not be able to use context-based completion because that needs to be provided by the application
- rlwrap example using msys2
```
pacman -S rlwrap
#Here is a nice configuration example for wrapping a command with rlwrap, the
#command in this example being sqlplus, this would go into your ~/.bashrc:

    alias sqlplus="rlwrap -i -f ~/.sqlplus_history -H ~/.sqlplus_history -s 30000 sqlplus --"
    touch ~/.sqlplus_history
```


### arguments

- pressing alt-number (can be multiple digit) before a command passes a numeric argument to the command
- by default it's repetition
- argument will be visible in prompt
- number can be negative, some commands interpret it as reversing the direction where the command is applied

### keybinding notation

`\m-k` - means alt+k
alternatively you can press esc, then immediately k (on keyboards without alt only?)
`\c-x s` - means ctrl - x then s

### keybindings

* editing
    * C-d - delete char
    * C-q or C-v - quoted insert - add keystrokes verbatim
    * Alt-TAB - insert tab
    * C-t - swap 2 chars
    * Alt-t - swap 2 words (Alt-ctrl-t is the same?)
    * Alt-u - uppercase word
    * Alt-l - lowercase word
    * Alt-c - capitalize word
    * Alt-# - insert comment
    * Alt-r - revert changes made to the line
    * C-_ or C-x C-u - undo
* mark - move cursor between 2 points and select text between them
    * C-@ - set mark to cursor
    * C-x C-x swap position with the mark and select text in between
* killing and yanking
    * killing deletes the text and adds it to the end of the killring
        * a sequence of kills without any other commands in between (including movement) is all appended together to make a single killring item
        * a kill puts the new value to the front (top) of the buffer
        * so order of items in the ring is last to first (unless rotated)
    * yanking copies items out of the killing
        * C-y - yank top item
        * M-y - replace last yanked item with the next one in the buffer and move the top of the buffer to that item
        * once rotated, the ring states that way
    * C-k - kill-line - from cursor to the end of the line
    * C-u - unix-line-discard - from cursor to beginning of the line (adds to killring)
    * Alt-d - kill-word
    * Alt-delete - backward kill word (doesn't work?)
    * Ctrl-w - backward kill word (space separated word)
    * Alt-C-y - yank the 1st (nth with argument) argument of previous history entry
        * todo: not sent properly by windows terminal?
    * Alt-./Alt-, - yank the last (nth with argument) of previous hitory entry
        * this one can be repeated to go backwards in history (numberic argument on repeated use specifies the direction) replacing previous press
* moving cursor
    * C-a - beginning of line
    * C-e - end of line
    * C-f - forward char
    * C-b - backward char
    * Alt-f - forward word
    * Alt-b - backward word
    * C-] then `<char>` - move cursor to next occurence of char
    * Alt-C-] then `<char>` - move cursor to prev occurance of char
* misc
    * C-l - clear screen
    * C-g - abort command
* "standard keybdings", some builtin some are custom:
    * left/right - move cursor a character
    * ctrl-left/right - move cursor a word
    * ctrl-del - delete word
    * ctrl-backspace - backward delete word
    * home/end - move cursor to beginning/end
    * up/down - move history
    * pgup/down - begin/end history?

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
    - up/down moves in the history
    - Alt-> to end Alt-< to beginning
    - Ctrl-o - accept current line + move to next one for editing, great for replaying history
- search
    - C-p/C-d moves to the previous/next line that starts with what's in the line before cursor (not by default but in my config)
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
    - entries preceded by whitespace aren't added to history

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

## editline - alternative readline api implementation

- it's a port of NetBSD's libedit, project's website: <http://thrysoee.dk/editline/>
    - windows implementation: <http://mingweditline.sourceforge.net/>
    - there's a different library with the same name which is based on an old fork of the same codebase: <https://github.com/troglobit/editline>
        - this fork doesn't support config files, but doesn't have the ncurses dependency
- configuration file has a different format compared to readline: <https://manpages.debian.org/testing/libedit-dev/editrc.5.en.html>
- documentation: <https://manpages.debian.org/testing/libedit2/editline.7edit.en.html>
- requirements
    - standard keybinds, configurable in a file - yes
    - possibility of integrating fzf - ?
        - registering custom commands - yes, EL_ADDFN, missing in mingweditline

## linenoise, now repl++

- minimalistic readline-like-functionality implementation, no readline api compatibility
- <https://github.com/AmokHuginnsson/replxx>
