# VSCode

- [educators documentation](https://code.visualstudio.com/learn/educators/python)
- [learn jupyter notebooks](https://code.visualstudio.com/learn/educators/notebooks)
- [microsoft vs code oss](https://github.com/microsoft/vscode/wiki/Differences-between-the-repository-and-Visual-Studio-Code)

## workspaces and sessions

- overview
    - <https://code.visualstudio.com/docs/editor/workspaces>
    - <https://code.visualstudio.com/docs/editor/multi-root-workspaces>
    - <https://code.visualstudio.com/docs/getstarted/settings>
    - <https://code.visualstudio.com/docs/getstarted/userinterface#_window-management>
- 3 types of workspaces
    - no workspace mode
        - opened by opening a single file when there's no active vscode, or with newwindow flag: `code -n filename`
        - purple status bar
        - no workspace settings editing available
        - "In this mode, some of VS Code's capabilities are reduced but you can still open text files and edit them"
        - the only way to restore the session is to reopen vscode with no arguments if no other files have been opened since
    - single root workspace mode
        - opened by opening a single dir (`code .`), will create `.vscode` dir if not already there
        - blue status bar
        - workspace-specific settings stored in `.vscode` in the root dir
        - opening the root will restore the last session for that workspace, even if files from outside were opened in that session
        - sessions are stored outside of .vscode dir and mapped to the project based on the opened path, so junctions/soft-links can confuse vscode into not loading the session
    - multi root workspace
        - opened by opening a `workspacfileedir/workspacename.code-workspace` file, opening multiple dirs at once, or by adding a second folder to an already running session
            - adding a folder can be done using file->add folder to workspace or using `code --add dir1 dir2` which will add dirs to the last active vscode instance
            - can also drag and drop a folder to be added
        - a workspace which doesn't have an explicit `.code-workspace` file is called an untitled workspace, with the session stored in a temporary `untitiled.code-workspace` file until the workspace is properly saved
        - blue status bar, the file explorer title has "(Workspace)" at the end 
        - workspace-specific settings stored in `workspacfileedir/workspacename.code-workspace` file
            - each root directory can have a `.vscode` dir, whose settings take priority over .code-workspace settings for the contents of that directory
            - global vscode settings from `.vscode` dirs are ignored to avoid conflict, and `.code-workspace` takes priority instead
            - any settings that are stored in `.vsode` dir can also be stored in `.code-workspace` including tasks, debug launch configs, etc.
        - `.code-workspace` files can be used to create projects for single directories and loose files too
            - allows for saving sessions/settings for loose files
            - allows for having multiple separate settings/sessions for the same set of files depending on which `.code-workspace` is used 
        - potentially useful for working on multiple projects at once, like in eclipse workspaces
        - tasks,files,searches,etc. that are present in multiple root directories will be disambigulated in the ui by showing the name of the root at the end of the item's name
        - a good way to work with projects which have nested projects
            - open master directory
            - add subdirectories with subprojects as additional root directories
            - save the workspace for later use
            - vscode will always open the files in the most nested subproject context even when opened from explorer window, so there's no duplicates of the same file
- command line
    - `code .` - open directory workspace 
    - `code filename` - open file in an active session, or create new no-workspace session
    - `code -r .` - open directory in the most recently used window, replacing previous session
    - `code -n` - open a new window
    - `code --diff` - diff viewer
    - `code -w` - wait until exit
    - `code --goto filepath:10:5` - go to line and character
    - `code --add dir1 dir2` add a folder or multiple folders to the last active VS Code instance for a multi-root workspace

## keybinds

- file explorer
    - preview mode
        - enabled using `workbench.editor.enablePreview*` setting
        - single click on a file opens a preview mode editor (filename displayed in italics)
        - if you open another file this way the new preview will replace the previous contents
        - double clicking or editing the file contents move the editor out of preview mode
    - alt+click opens in a split editor grup
    - drag file from explorer(or other file icon) to the editor group area
        - hover over centre to open
        - hover over edge area to open in a split editor group
    - filter on type
        - typing while explorer is focused to filter the view
    - by default explorer excludes contents of .gitignore, some standard dirs like .git and contents of files.exclude setting 
    - ctrl/shift click - standard multiple selection
- editor
    - ctrl+z - undo
    - ctrl+u - undo cursor position change
    - ctrl+shift+z - redo
    - ctrl+f/h - find/replace
        - with regext mode (alt+r) parenthesis define groups and $num refers to parenthesis group in the repla mode
        - (shift)+f3 - find (previous)next
        - alt+enter - select all occurences of the match
    - ctrl+shift+a - select all
    - shift+alt+left/right - shrink/expand selection
    - ctrl+s - save
    - alt+z - toggle word wrap
    - alt+f5 - next change in diff view
    - alt+left/right - navigate to previous/next location in editors
    - multiple cursors
        - alt+click - add multiple cursors with each click
        - ctrl+alt+up/down - multiple cursor to line above/below
        - ctrl+shift+l - multiple cursors to all occurences of selection
        - ctrl+d - add cursor to next occurence of selection (can be pressed multiple times)
        - shift+alt+mouse - select column block of text, adds multiple cursors to the end of the line
    - manipulate line
        - shift+alt+up/down - copy current line to above/below
        - alt+up/down - move current line to above/below
        - ctrl+l - select current line
        - ctrl+k - delete line
        - ctrl+enter - insert line above
        - ctrl+shift+enter - insert line below
    - markdown
        - ctrl+shift+v - open markdown preview
        - ctrl+k v - side by side markdown preview
    - folding - good for hiding stuff that gets in the way
        - ctrl+shift+`[`/`]` - fold/unfold at the cursor
        - ctrl+k ctrl+l - toggle fold of the region under cursor
        - ctrl+k ctrl+`[`/`]` - fold/unfold recursively current region and regions inside that region
        - ctrl+k ctrl+0 - fold all
        - ctrl+k ctrl+`<number>` - fold all regions of level n *except the region with the cursor currently* (including subregions)
            - ctrl+k ctrl+1 - fold only level one stuff (the most convenient variant)
        - ctrl+k ctrl+j - unfold all
        - ctrl+k ctrl+/ - fold all block comments
    - language support
        - ctrl+space - trigger suggestion
        - ctrl+shift+space - trigger parameter hints
        - ctrl+. - quick fix
        - ctrl+/ - toggle line comment
        - Ctrl+K Ctrl+C - Add line comment
        - Ctrl+K Ctrl+U - Remove line comment
        - shift+alt+a - toggle block comment
        - f12 - go to symbol definition
            - alt+f12 - peek a symbol
            - shift+f12 - go to references
            - shift+alt+f12 - find all references
        - f2 - renme selected symbol
    - formatting
        - ctrl+k ctrl+x - trim trailing whitespace
        - ctrl+k ctrl+f - format selection
        - shift+alt+f - format file
    - standard keybinds:
        * ctrl-left - move to beginning of prev word
        * ctrl-right - move to end of next word
        * ctrl-backspace - delete prev word
        * ctrl-delete - delete next word
        * ctrl-up - scroll up a line
        * ctrl-down - scroll down a line
        * home/end - beginning/end of current line
        * ctrl-home/end - beginning/end of the text
        * pg up/down - move cursor by a page
        * ctrl-pgup/pgdown - scroll page up/down
        * shift-left/right - select char at a time
        * ctrl-shift-left/right - select word at a time
        * shift-ctrl-home/end - select text towards beginning/end
        * shift-pgup/down - select a page
- command pane:
    - ? shows different action types
    - type command + space to do an action:
        - `term ` switch to chosen terminal
        - `> ` commands (ctrl+shift+p)
        - `@ ` go to symbol (ctrl+shift+o)
            - `@: ` - groups symbols by type
        - `# ` go to symbol in workspace (ctrl+t)
        - `ext ` go to extensions (ctrl+shift+x)
        - `debug ` run debug configuration
        - `: ` go to line (ctrl+g)
        - `view ` quick open view (ctrl+q)
        - `edt ` switch editors in tab order (alt+pgup/pgdown)
        - `edt mru ` switch editors in mru order in all groups
        - `edt active ` switch editors in mru order in current group (ctrl+(shift)+tab)
        - `task ` run tasks
    - the pane will stay up while holding ctrl-hotkey
    - clicking entry with `alt` will force same window, `ctrl` new window
    - ctrl+enter on an entry will open in a new editor group
    - you can select multiple entries by pressing arrow key multiple times on each entry
- global keybinds (anywhere while not in terminal)
    - alt + mouse scroll - 5x faster scrolling
    - ctrl+alt+z - reopen closed editor
    - ctrl+j - toggle bottom panel
    - ctrl+b - toggle sidepanel
    - ctrl+q - quick open view (change focused pane)
    - ctrl+p - quick open (open file from workspace by name, doesn't need to be open)
    - ctrl+n - open new file
    - ctrl+o - open file from fs dialog
    - shift+alt+r - open file in explorer
    - ctrl+g - go to line
    - ctrl+r - open recent workspace in this window (ctrl+click opens a new window)
    - ctrl+w/ctrl+f4 - close editor pane
    - ctrl+shift+o - go to symbol
    - ctrl+t - go to symbol in workspace
    - ctrl+shift+s - save as
    - ctrl+shift+h - replace in files
    - ctrl+shift+f - find in files
    - ctrl+shift+./; - open/focus on breadcrumbs - go to breadcrumb in file/parents
    - switching editors
        - alt+`<num>` - go to nth editor of the current group
        - ctrl-9 - go to last editor (in tab order) of the current group
        - ctrl-<1-8> - go to (or create) nth editor group
        - alt-pgup/pgdown - next/prev editor in the tab order
        - ctrl+shift+pgup/pgdown - move editor in the open editors list and tabs (within current group)
        - ctrl+(shift)+tab - switch editors in mru order in current group
        - ctrl+shift+p -> `edt `
    - editor groups
        - view -> editor layout menu has predefined editor layouts
            - "single" - merges all editor groups
            - grid - 2x2 layout
        - you can create editor groups by dragging files/editors towards edges of the editor area
            - this will move the editor to a new group
        - open editors view shows all groups
        - shift+alt+1, shift+alt+9 - move editor to first/last group (does nothing if there's one group)
        - ctrl+alt+left, ctrl+alt+right - move editor to prev/next group
        - ctrl+\ - split editor - copy editor to a new group
        - shift+alt+0 - when there are multiple editor groups, switch splitting between horizontal and vertical
- global keybinds (anywhere including terminal)
    - ctrl-shift-p - command pane
    - f11 - fullscreen
    - debug shortcuts: alt+f5, shift+alt+f5, ctrl+shift+f5, ctrl+f5, f5, shift+f11, f10, shift+f5, f6, f2
    - ctrl-shift-w - close window
    - ctrl-shift-n - new window
    - f6, shift-f6 - focus to next/previous "part"(ui area) 
    - ctrl-<1-8> - focus to (or create) nth editor group
    - ctrl-0 - focus on side bar
    - ctrl-9 - focus to last editor (in tab order) of the current group
    - alt-pgup/pgdown - next/prev editor in the tab order
    - ctrl+(shift)+tab - switch editors in mru order in current group
    - ctrl-shift-q - quick open view (go backwards in the list on repeated press)
    - ctrl+shift+t - focus to terminal/open new terminal
    - ctrl+shift+r - focus on the active editor
    - ctrl-shift-d(debug)/e(explorer)/x(extensions)/g(source control)/u(output pane)/m(problems pane)/y(debug console)/t(terminal) - focus switch
    - ctrl-shift-b - build task
    - ctrl-shift-plus - enable tab focus change
- terminal
    - alt+down/up/left/right - pane focus control
    - standard terminal scroll:
          scrollDown - "ctrl+down"
          scrollDownPage - "ctrl+pgdn"
          scrollUp - "ctrl+up"
          scrollUpPage - "ctrl+pgup"
          scrollToTop - "ctrl+home"
          scrollToBottom - "ctrl+end"
    - alt-shift-d - duplicate pane
    - ctrl-shift-c/v - copy-paste
    - shift-alt-c - copy current cwd of the terminal
    - (shift-)f3 - (backward)-find next
    - escape (clear selection)
    - ctrl+shift+t - new terminal
    - ctr+shift+f - find
    - ctrl+shift+q - quick open view (switch between panes)
    - ctrl+k ctrl+k - ctrl+k

## setup/configuration - windows

- add code to ALL path variable
- terminal
    - unbind all global keybindings which interfere with terminal
    - bind all navigation keybindings to key shortcuts terminal doesn't care about (ctrl-shift, ctrl-num) or bind them only when not in terminalFocus
    - use cycle-settings to add a mode which enables terminal getting all the input
- accessing workspaces/sessions
    - use project manager extension to view single-dir workspaces
        - supports base search directories and favourites (stored custom path)
    - use workspace explorer extension to view .code-workspace workspaces
        - stored in a base dir and subdirs
- install bookmarks extension
- install windows explorer context menu
- install [vscode-neovim](https://marketplace.visualstudio.com/items?itemName=asvetliakov.vscode-neovim)
    - disable scrolling past end
    - enable always starting with insert mode
```vim
if exists('g:vscode')
" start in insert mode
    au BufEnter * start
endif
```
    - disable insert mode keybindings which overlap with desired vscode keybinds
```json
    // disable keybinds that are duplicates of vscode binds or interfere with standard vscode keybinds
    { "key": "ctrl+c",                "command": "-vscode-neovim.escape" },
    { "key": "ctrl+oem_4" ,                "command": "-vscode-neovim.escape" },
    { "key": "f1",                    "command": "-vscode-neovim.send" },
    { "key": "shift+win+p",           "command": "-vscode-neovim.send"},
    { "key": "shift+win+o",           "command": "vscode-neovim.send"},
    
    { "key": "ctrl+j",                "command": "-editor.action.insertLineAfter"}, // this one overrides toggle panel, so it's fine?, but then ctrl+enter is very easy to type too
    { "key": "ctrl+d",                "command": "-editor.action.outdentLines"},
    { "key": "ctrl+t",                "command": "-editor.action.indentLines"},
    { "key": "ctrl+h",                "command": "-deleteLeft"},
    { "key": "ctrl+w",                "command": "-deleteWordLeft"},
    { "key": "ctrl+u",                "command": "-deleteAllLeft"},
    // rebind vscode keybindings that conflict with useful neovim insert keybinds
    { "key": "ctrl+shift+a",                "command": "editor.action.selectAll"}, // conficted with ctrl+a
    { "key": "ctrl+r ctrl+r",              "command": "workbench.action.openRecent", // conflicted with ctrl+r global binding
        "when": "editorTextFocus && neovim.ctrlKeysInsert && !neovim.recording && neovim.mode == 'insert'"},
```
- todo: some templates for workspace configuration
- todo: fuzzyfind? https://github.com/rlivings39/vscode-fzf-quick-open

### setup linux

- running `"C:\Users\qasto\AppData\Local\Programs\Microsoft VS Code\bin\code"` from linux runs windows vscode, first time run installs a connector to windows
- official binary - better support for most features
pacaur -S pacaur -S visual-studio-code-bin 
add export DONT_PROMPT_WSL_INSTALL=1 to .bashrc
- alternatively OSS - slightly broken (some features don't work)
```
pacman -S code
pacaur -S code-marketplace code-features
```

## interesting settings extensions

- settings sync allows syncing config files
    - each part of the config can be commented out and disabled on a machine: <https://github.com/shanalikhan/code-settings-sync/wiki/Sync-Pragmas>
- settings cycler can be used to toggle a setting with a keybinding
  - useful to trigger terminal friendly mode for example
  - sadly, doesn't have a builtin way to display setting in the status bar, but this can be emulated by changing settings of some extension or ui
     - who's the boss and project tag extensions allow showing status
     - could also change a ui setting, like colour of terminal or status bar
     - todo: make an extension fork that does these things and also adds a picker option for cycling?
  - the override workspace settings setting only overrides workspace entry if workspace has a config for the values already :(
- multi-command - bind multiple commands to a single command name
- commands - allows binding any command to status bar and to display custom text (sadly doesn't evaluate variables)
- differently colored windows for easy distinguishing: https://marketplace.visualstudio.com/items?itemName=johnpapa.vscode-peacock
- remote container development plugin automatically starts a container defined in .devcontainer directory/file - use remote container open directory command
    - vscode installs a remote module in the container, similar to wsl, and executes plugins there
- there's an option to just attach to an existing container and also configure steps vscode would do automatically for that container
    - <https://code.visualstudio.com/docs/remote/attach-container>

### vim integration

- vscode vim reimplements vim in vscode
    - [supports more vim special keys in insert mode](https://github.com/VSCodeVim/Vim/blob/master/ROADMAP.md#insert-mode-keys)
        - doesn't support all of them, but supports CTR+O (switch to normal for a single command)
    - configuring to work well with vscode keybindings:
        - on the vscode level remap all ctrl bindings:
            - leave regular mapping for `vim.mode != 'Insert'
            - add ctrl-; `<key>` mapping for insert (or all) modes so that they don't conflict with default mappings and insert works like standard vscode editor
- vscode neovim connects to neovim for modes other than insert
    - [less support for insert mode keys](https://github.com/asvetliakov/vscode-neovim#insert-mode-special-keys)
        - [doesn't support ctrl+o](https://github.com/asvetliakov/vscode-neovim/issues/181#issuecomment-585264621)
        - fixed ctrl+o on my local branch
    - superior to vscode vim in every other way
    - custom insert mode mappings from neovim won't work, configre these in keybindings.json instead
    - modes different than normal are displayed in the vscode status bar
    - visual mode selections are displayed on vscode but aren't always accurate, and aren't real vscode selections
        - to make them a real selection press ctrl+shift+p, as a bonus you can then do a command on the selection or just escape
    - keyboard events are sent to vim only when forwarded through [keybindings.json](https://github.com/asvetliakov/vscode-neovim#pass-additional-keys-to-neovim-or-disable-existing-ctrl-keys-mappings)
        - [list of passed ctrl keymaps](https://github.com/asvetliakov/vscode-neovim#normal-mode-control-keys)
    - custom normal mode mappings [should be done from neovim](https://github.com/asvetliakov/vscode-neovim#custom-keymaps-for-scrollingwindowtabetc-management)
        - [all bindings](https://github.com/asvetliakov/vscode-neovim/tree/master/vim)

### c++

- [default cpp extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools)
    - provides support for build tasks, debugging and code navigation/analysis using internal intellisense engine
    - can be configured to use any toolchain as long as you set all the variables
- [cmake tools](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cmake-tools)
    - tools for running build configuration and running cmake generation
    - it's a build config provider for the cpp extension
- [msys/mingw toolchains](https://marketplace.visualstudio.com/items?itemName=fougas.msys2)
    - provides easy to use configuration variables for msys2 and mingw and example configuration to plug these into the default c++ plugin and cmake tools
- [cmake](https://marketplace.visualstudio.com/items?itemName=twxs.cmake)
    - cmake text editing support (completion and stuff)
- clang tools using compile_commands.json
    - [ccls](https://marketplace.visualstudio.com/items?itemName=ccls-project.ccls)
        - lsp server based navigation/analysis, runs configured executable in the configured location and provides completion based on the actual commands used in the build (using `compile_commands.json`)
        - when used disable default cpp navigation analysis feature
        - [example usage - aether](https://github.com/hadeaninc/aether/blob/develop/EDITORS.md#vscode)
    - [clangd](https://marketplace.visualstudio.com/items?itemName=llvm-vs-code-extensions.vscode-clangd)
        - same as ccls, but uses clangd
    - [clang format](https://marketplace.visualstudio.com/items?itemName=xaver.clang-format)
    - [clang tidy gui](https://marketplace.visualstudio.com/items?itemName=TimZoet.clangtidygui)
    - [clang tidy problems list](https://marketplace.visualstudio.com/items?itemName=notskm.clang-tidy)
    - [include what you use](https://marketplace.visualstudio.com/items?itemName=pokowaka.pokowaka-iwyu)
        - header cleanup
- [lldb integration](https://marketplace.visualstudio.com/items?itemName=vadimcn.vscode-lldb)
    - the default extension doesn't support lldb on windows/linux
- [c++ advanced lint](https://marketplace.visualstudio.com/items?itemName=jbenden.c-cpp-flylint)
    - runs static analysis tools that you have in your path
- [alternative cmake integration](https://marketplace.visualstudio.com/items?itemName=go2sh.cmake-integration-vscode)
    - less updated
    - 1 step build, similar to editors which read cmake directly

### Markdown

Outline in explorer shows headers

Plugins:

- Markdown Paste - pasting images from clipboard
- markdown shortcuts - formatting stuff
- markdown checkbox - checkbox support

Preview: RMB -> open in preview

## wsl

- make sure to run code.sh from windows path for starting windows wsl integration or linux path with windows path removed to guarantee linux startup

## config files

- settings:
```json
{
    "breadcrumbs.enabled": true,
    "editor.renderWhitespace": "none",
    "projectManager.groupList": true,
    "workspaceExplorer.workspaceStorageDirectory": "C:\\Users\\Dariusz\\localworkspace\\vsworkspaces",
    "workspaceExplorer.additionalCustomIconDirectory": "C:\\Users\\Dariusz\\localworkspace\\vsworkspaces\\icons",
    "editor.fontFamily": "Consolas, 'Courier New', monospace",
    "terminal.external.windowsExec": "wt",

    
    "workbench.editorAssociations": [
        {
            "viewType": "jupyter.notebook.ipynb",
            "filenamePattern": "*.ipynb"
        }
    ],
    "commands.commands": [ // Array of commands
            { // An object describing a command, most entries are optional
              "alignment": "left", // Should the item be placed to the left or right?
              "priority": 0, // The priority of this item. Higher value means the item should be shown more to the left
              "color": "#dfdfdf", // The foreground color for this item
              "text": "$(terminal) Toggle Shell", // The text to show for the entry
              "command": "settings.cycle.switchTerminalWindows", // Command to execute
            },
            { // An object describing a command, most entries are optional
                "alignment": "left", // Should the item be placed to the left or right?
                "priority": 0, // The priority of this item. Higher value means the item should be shown more to the left
                "color": "#dfdfdf", // The foreground color for this item
                "text": "$(terminal) Focus Mode", // The text to show for the entry
                "command": "settings.cycle.switchTerminalFocus", // Command to execute
            },
    ],

    // terminal settings
    "terminal.integrated.fontWeight": "1",
    "terminal.integrated.fontWeightBold": "5",
    "terminal.integrated.fontFamily": "'Delugia Mono Nerd Font'",
    "terminal.integrated.scrollback": 2000,
    "terminal.integrated.rightClickBehavior": "copyPaste",
    "terminal.integrated.commandsToSkipShell": [],
    "terminal.integrated.allowMnemonics": false,
    "terminal.integrated.copyOnSelection": false,
    // shell used by tasks and debug
    "terminal.integrated.automationShell.windows": "C:\\Program Files\\PowerShell\\7\\pwsh.exe",
    // switching settings
    // defaults for "switchTerminalWindows"
    "StatusBarTag.enable": true,
    "terminal.integrated.shell.windows": "C:\\portable\\bin\\winshell\\bash-mingw64-login.exe",
    "StatusBarTag.text": "$(terminal) mingw64",
    // default for switchTerminalFocus
    "terminal.integrated.sendKeybindingsToShell": false, // overrides commandsToSkipShell to not allow any (and nothing else)
    "terminal.integrated.allowChords": true,
    "terminal.integrated.altClickMovesCursor": true,
    "workbench.colorCustomizations": {
        "panel.background": "#202020"
    },
    // switches
    "settings.cycle": [
        {
            "id": "switchTerminalWindows",
            "overrideWorkspaceSettings": true,
            "values": [
                {
                    "terminal.integrated.shell.windows": "C:\\Windows\\System32\\cmd.exe",
                    "StatusBarTag.text": "$(terminal) cmd",
                },
                {
                    "terminal.integrated.shell.windows": "C:\\Program Files\\PowerShell\\7\\pwsh.exe",
                    "StatusBarTag.text": "$(terminal) pwsh",
                },
                {
                    "terminal.integrated.shell.windows": "C:\\portable\\bin\\winshell\\bash-mingw64-login.exe",
                    "StatusBarTag.text": "$(terminal) mingw64",
                },
                {
                    "terminal.integrated.shell.windows": "C:\\portable\\bin\\winshell\\bash-msys-login.exe",
                    "StatusBarTag.text": "$(terminal) msys",
                },
                {
                    "terminal.integrated.shell.windows": "C:\\Windows\\System32\\wsl.exe",
                    "StatusBarTag.text": "$(terminal) wsl",
                },
                {
                    "terminal.integrated.shell.windows": "C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe",
                    "StatusBarTag.text": "$(terminal) powershell",
                },
            ]
        },
        {
            "id": "switchTerminalFocus",
            "overrideWorkspaceSettings": true,
            "values": [
                {
                    "workbench.colorCustomizations": {
                        "panel.background": "#202020",
                    },
                    "terminal.integrated.sendKeybindingsToShell": false,
                    "terminal.integrated.allowChords": true,
                    "terminal.integrated.altClickMovesCursor": true,
                    "terminal.integrated.commandsToSkipShell": [],
                },
                {
                    "workbench.colorCustomizations": {
                        "panel.background": "#111111",
                    },
                    "terminal.integrated.sendKeybindingsToShell": true,
                    "terminal.integrated.allowChords": false,
                    "terminal.integrated.altClickMovesCursor": false,
                },
            ]
        },
    ],
    "projectManager.git.baseFolders": [
        "C:\\Users\\qasto\\localworkspace"
    ],
    "projectManager.git.maxDepthRecursion": 1,
    "projectManager.hg.baseFolders": [
        "C:\\Users\\qasto\\localworkspace"
    ],
    "projectManager.hg.maxDepthRecursion": 1,
    "projectManager.removeCurrentProjectFromList": false,
    "projectManager.svn.maxDepthRecursion": 1,
    "projectManager.vscode.baseFolders": [
        "C:\\Users\\qasto\\localworkspace"
    ],
    "projectManager.vscode.maxDepthRecursion": 1,
}
```

- keybinds:
```json
// Place your key bindings in this file to override the defaults
[
    // terminal launching block
    {
        "key": "ctrl+oem_3",
        "command": "-workbench.action.terminal.toggleTerminal"
    },
    {
        "key": "ctrl+shift+t",
        "command": "-workbench.action.reopenClosedEditor"
    },
    {
        "key": "ctrl+alt+z",
        "command": "workbench.action.reopenClosedEditor",
        "when": "!terminalFocus"
    },
    {
        "key": "ctrl+shift+t",
        "command": "workbench.action.terminal.toggleTerminal",
        "when": "!terminalFocus"
    },
    {
        "key": "ctrl+shift+t",
        "command": "workbench.action.terminal.new",
        "when": "terminalFocus"
    },
    // bind keys to go back to editor
    {
        "key": "ctrl+shift+r",
        "command": "workbench.action.focusActiveEditorGroup",
        "when": "!editorTextFocus && !inSearchEditor && !inQuickOpen"
    },
    // scrolling block
    // "standard scrolling bindings
    {
        "key": "ctrl+end",
        "command": "workbench.action.terminal.scrollToBottom",
        "when": "terminalFocus && terminalProcessSupported"
    },
    {
        "key": "ctrl+home",
        "command": "workbench.action.terminal.scrollToTop",
        "when": "terminalFocus && terminalProcessSupported"
    },
    {
        "key": "ctrl+down",
        "command": "workbench.action.terminal.scrollDown",
        "when": "terminalFocus && terminalProcessSupported"
    },
    {
        "key": "ctrl+up",
        "command": "workbench.action.terminal.scrollUp",
        "when": "terminalFocus && terminalProcessSupported"
    },
    {
        "key": "ctrl+pagedown",
        "command": "workbench.action.terminal.scrollDownPage",
        "when": "terminalFocus && terminalProcessSupported && !terminalAltBufferActive"
    },
    {
        "key": "ctrl+pageup",
        "command": "workbench.action.terminal.scrollUpPage",
        "when": "terminalFocus && terminalProcessSupported && !terminalAltBufferActive"
    },
    // unbind unnecessary bindings
    {
        "key": "ctrl+alt+pagedown",
        "command": "-workbench.action.terminal.scrollDown",
        "when": "terminalFocus && terminalProcessSupported"
    },
    {
        "key": "ctrl+alt+pageup",
        "command": "-workbench.action.terminal.scrollUp",
        "when": "terminalFocus && terminalProcessSupported"
    },
    // remap keys conflicting with "standard scrolling bindings"
    {
        // ctrl to alt
        "key": "alt+pagedown",
        "command": "workbench.action.nextEditor"
    },
    {
        "key": "ctrl+k ctrl+pagedown",
        "command": "workbench.action.nextEditorInGroup"
    },
    {
        "key": "alt+pageup",
        "command": "workbench.action.previousEditor"
    },
    {
        "key": "ctrl+k ctrl+pageup",
        "command": "workbench.action.previousEditorInGroup"
    },
    // now that ctrl-pageup/down is free these can be remapped too
    {
        "key": "ctrl+pagedown",
        "command": "scrollPageDown",
        "when": "textInputFocus"
    },
    {
        "key": "ctrl+pageup",
        "command": "scrollPageUp",
        "when": "textInputFocus"
    },
    // free ascii non-printing character block for keybinds block
    // send ctrl-k (have to press twice, but it's a good enough compromise)
    {
        "key": "ctrl+k ctrl+k",
        "command": "workbench.action.terminal.sendSequence",
        "when": "terminalFocus",
        "args": {
            "text": "\u000b"
        }
    },
    // fix a bad rebind vscode does specificly for cmd for some dumb reason
    {
        "key": "ctrl+backspace",
        "command": "workbench.action.terminal.sendSequence",
        "when": "terminalFocus && terminalShellType == 'cmd'",
        "args": {
            "text": "\u0017"
        }
    },
    // unbind the global shortcuts in terminal (shortcuts which don't have when clause) so that the terminal input isn't intercepted
    //  - it looks like "ctrl-y redo" don't interfere with terminal handling of ctrl-y yank because they work only in text contexts?
    //  - but workbench actions like switch pane do interfere
    //  - sadly this needs to be checked on case-by-case basis if the editor action is triggered while terminal is in focus
    //  - actions listed in terminal.integrated.commandsToSkipShell are going to intercept termial unless "terminal.integrated.sendKeybindingsToShell" is set
    //  - "terminal.integrated.sendKeybindingsToShell" = doesn't affect all keys, maybe it only cares about keybinds that are producing escape codes in terminal?
    // integrated terminal keybindings (intercept terminal input):
    //      - global keybinds (anywhere including terminal)
    //         - ctrl-shift-p - command pane
    //         - f11 - fullscreen
    //         - debug shortcuts: alt+f5, shift+alt+f5, ctrl+shift+f5, ctrl+f5, f5, shift+f11, f10, shift+f5, f6, f2
    //         - ctrl-shift-w - close window
    //         - ctrl-shift-n - new window
    //         - f6, shift-f6 - focus to next/previous "part"(ui area) 
    //         - ctrl-<1-8> - focus to (or create) nth editor group
    //         - ctrl-0 - focus on side bar
    //         - ctrl-9 - focus to last editor (in tab order) of the current group
    //         - alt-pgup/pgdown - next/prev editor in the tab order
    //         - ctrl+(shift)+tab - switch editors in mru order in current group
    //         - ctrl-shift-q - quick open view (go backwards in the list on repeated press)
    //         - ctrl+shift+t - focus to terminal/open new terminal
    //         - ctrl+shift+r - focus on the active editor
    //         - ctrl-shift-d(debug)/e(explorer)/x(extensions)/g(source control)/u(output pane)/m(problems pane)/y(debug console)/t(terminal) - focus switch
    //         - ctrl-shift-b - build task
    //         - ctrl-shift-plus - enable tab focus change
    //     - terminal
    //         - alt+down/up/left/right - pane focus control
    //         - standard terminal scroll:
    //               scrollDown - "ctrl+down"
    //               scrollDownPage - "ctrl+pgdn"
    //               scrollUp - "ctrl+up"
    //               scrollUpPage - "ctrl+pgup"
    //               scrollToTop - "ctrl+home"
    //               scrollToBottom - "ctrl+end"
    //         - alt-shift-d - duplicate pane
    //         - ctrl-shift-c/v - copy-paste
    //         - shift-alt-c - copy current cwd of the terminal
    //         - (shift-)f3 - (backward)-find next
    //         - escape (clear selection)
    //         - ctrl+shift+t - new terminal
    //         - ctr+shift+f - find
    //         - ctrl+shift+q - quick open view (switch between panes)
    //         - ctrl+k ctrl+k - ctrl+k
    // confirmed non-intercepted inputs with global binds (probably not listed in "terminal.integrated.commandsToSkipShell")
    //  - ctrl-a, ctrl-z, ctrl-shift-z, alt+f1, ctrl-h, alt-z, f12, esc?, ctrl+shift+., ctrl+shift+;, shift+alt+r, ctrl+w, ctrl+f4, alt+f5, ctrl-n, ctrl-o
    //  - ctrl-s, ctrl+g, ctrl+shift+pageup, ctrl+shift+pagedown, shift+alt+9, shift+alt+1, ctrl+alt+left, ctrl+alt+right, alt-left, alt-right, alt-<1-9>, ctrl-r, ctrl-t,
    //  - ctrl-oem5(\), shift+alt+0, ctrl+shift+f
    // unbind redundant keys
    // alt+0, redundant with ctrl-9, other alt+ aren't active in terminal
    {
        "key": "alt+0",
        "command": "-workbench.action.lastEditorInGroup"
    },
    // ctrl-e - duplicate of ctrl-p
    {
        "key": "ctrl+e",
        "command": "-workbench.action.quickOpen"
    },
    // f1 is a duplicate of ctrl-shift-p
    {
        "key": "f1",
        "command": "-workbench.action.showCommands"
    },
    // essentially a duplicate of ctrl-shift-w
    {
        "key": "ctrl+w",
        "command": "-workbench.action.closeWindow"
    },
    // dup of ctrl-pgup/down
    {
        "key": "shift+pagedown",
        "command": "-workbench.action.terminal.scrollDownPage"
    },
    {
        "key": "shift+pageup",
        "command": "-workbench.action.terminal.scrollUpPage"
    },
    // let terminal handle copy and paste
    {
        "key": "ctrl+c",
        "command": "-workbench.action.terminal.copySelection"
    },
    {
        "key": "ctrl+v",
        "command": "-workbench.action.terminal.paste"
    },
    {
        "key": "ctrl+v",
        "command": "-workbench.action.terminal.sendSequence"
    },
    // rebind to wt keys
    {
        "key": "ctrl+shift+5",
        "command": "-workbench.action.terminal.split"
    },
    {
        "key": "alt+shift+d",
        "command": "workbench.action.terminal.split",
        "when": "terminalFocus && terminalProcessSupported"
    },
    {
        "key": "ctrl+f",
        "command": "-workbench.action.terminal.focusFind"
    },
    {
        "key": "ctrl+shift+f",
        "command": "workbench.action.terminal.focusFind",
        "when": "terminalFindFocused && terminalProcessSupported || terminalFocus && terminalProcessSupported"
    },
    // rebind in terminal, but leave binding for outside of terminal
    {
        "key": "ctrl+q",
        "command": "-workbench.action.quickOpenView"
    },
    {
        "key": "ctrl+shift+q",
        "command": "workbench.action.quickOpenView",
        "when": "!inQuickOpen"
    },
    {
        "key": "ctrl+q",
        "command": "workbench.action.quickOpenView",
        "when": "!terminalFocus && !inQuickOpen"
    },
    // rebind to exclude terminal
    // todo: replace some of these by removing these from "terminal.integrated.commandsToSkipShell" (or adding these with - in front)
    {
        "key": "ctrl+shift+s",
        "command": "-workbench.action.files.saveAs"
    },
    {
        "key": "ctrl+shift+s",
        "command": "workbench.action.files.saveAs",
        "when": "!terminalFocus"
    },
    {
        "key": "ctrl-p",
        "command": "-workbench.action.quickOpen"
    },
    {
        "key": "ctrl-p",
        "command": "workbench.action.quickOpen",
        "when": "!terminalFocus && !inFilesPicker"
    },
    {
        "key": "ctrl+shift+o",
        "command": "-workbench.action.gotoSymbol"
    },
    {
        "key": "ctrl+shift+o",
        "command": "workbench.action.gotoSymbol",
        "when": "!terminalFocus"
    },
    {
        "key": "ctrl+shift+h",
        "command": "-workbench.action.replaceInFiles"
    },
    {
        "key": "ctrl+shift+h",
        "command": "workbench.action.replaceInFiles",
        "when": "!terminalFocus"
    },
    {
        "key": "ctrl+b",
        "command": "-workbench.action.toggleSidebarVisibility"
    },
    {
        "key": "ctrl+b",
        "command": "workbench.action.toggleSidebarVisibility",
        "when": "!terminalFocus"
    },
    {
        "key": "ctrl+j",
        "command": "-workbench.action.togglePanel"
    },
    {
        "key": "ctrl+j",
        "command": "workbench.action.togglePanel",
        "when": "!terminalFocus"
    },
    {
        "key": "ctrl+shift+f",
        "command": "-workbench.action.findInFiles"
    },
    {
        "key": "ctrl+shift+f",
        "command": "workbench.action.findInFiles",
        "when": "!terminalFocus"
    },
    {
        "key": "ctrl+r",
        "command": "-workbench.action.reloadWindow",
        "when": "isDevelopment"
    },
    {
        "key": "ctrl+r",
        "command": "workbench.action.reloadWindow",
        "when": "!terminalFocus && isDevelopment"
    },
    {
        "key": "ctrl+shift+i",
        "command": "workbench.action.toggleDevTools",
        "when": "!terminalFocus && isDevelopment"
    },
    {
        "key": "f10",
        "command": "-extension.node-debug.startWithStopOnEntry"
    },
    {
        "key": "f10",
        "command": "extension.node-debug.startWithStopOnEntry",
        "when": "!terminalFocus && !inDebugMode && debugConfigurationType == 'node'"
    },
    {
        "key": "ctrl+m",
        "command": "-editor.action.toggleTabFocusMode"
    },
    {
        "key": "ctrl+shift+oem_plus",
        "command": "editor.action.toggleTabFocusMode"
    },
    // not needed shortcut block
    {
        "key": "ctrl+shift+oem_period",
        "command": "-breadcrumbs.toggleToOn"
    },
    {
        "key": "ctrl+oem_comma",
        "command": "-workbench.action.openSettings"
    },
    {
        "key": "ctrl+numpad_add",
        "command": "-workbench.action.zoomIn"
    },
    {
        "key": "ctrl+shift+oem_plus",
        "command": "-workbench.action.zoomIn"
    },
    {
        "key": "ctrl+oem_plus",
        "command": "-workbench.action.zoomIn"
    },
    {
        "key": "ctrl+numpad_subtract",
        "command": "-workbench.action.zoomOut"
    },
    {
        "key": "ctrl+shift+oem_minus",
        "command": "-workbench.action.zoomOut"
    },
    {
        "key": "ctrl+oem_minus",
        "command": "-workbench.action.zoomOut"
    },
    {
        "key": "ctrl+numpad0",
        "command": "-workbench.action.zoomReset"
    },
    {
        "key": "shift+alt+p",
        "command": "-projectManager.listProjects"
    },
    {
        "key": "ctrl+shift+f9",
        "command": "-o.showOutput"
    },
]
```
