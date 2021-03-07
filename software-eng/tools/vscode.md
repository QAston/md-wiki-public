# VSCode

## workspaces and projects

- Opening a file
    - will open an editor in open editors section, but will not add the file to the project
    - if a project is already opened (o), it will open in the project open editors, but the file is still independent
- Opening a directory
    - will add a "project" view in explorer, showing contents of the directory
    - will potentially add `.vscode` dir in the directory, settings in that directory will apply to the contents of the directory
    - project, (root) directory and (root) folder are synonymous in this context
    - projects are the unit of management/execution/settings etc.
    - opening another directory from file-> open will close current "project" and open a new window
    - there's a project manager plugin which can scan directories for projects and provide a list of saved projects to easily switch between them
- Opening a workspace
    - workspace is a group of (root) directories/folderes opened in a single vscode window
    - workspaces are optional, they are only mandatory for multiple projects in a single window
    - workspace is represented by a `*.code-workspace` file
    - every project in a workspace gets it's own `.vscode` directory with settings applied only to that root directory
    - to create a workspace:
        - file -> add folder to workspace, then save a workspace file when prompted
        - file -> save workspace as will create a workspace file if there isn't one
    - for workspace explorer plugin it's neccessary to put workspace files in a single directory

## Markdown

Outline in explorer shows headers

Plugins:

- Markdown Paste - pasting images from clipboard
- markdown shortcuts - formatting stuff
- markdown checkbox - checkbox support

Preview: RMB -> open in preview

## vscodevim

- emulates vim
- disable ctrl-keys
- on the vscode level remap all ctrl bindings:
    - leave regular mapping for `vim.mode != 'Insert'
    - add ctrl-; `<key>` mapping for insert (or all) modes so that they don't conflict with defaults
```
// Place your key bindings in this file to override the defaults
[
{ "key": "ctrl+; a",                "command": "extension.vim_ctrl+a",
        "when": "editorTextFocus && vim.active  && !inDebugRepl" },
{ "key": "ctrl+; b",                "command": "extension.vim_ctrl+b",
        "when": "editorTextFocus && vim.active  && !inDebugRepl && vim.mode != 'Insert'" },
{ "key": "ctrl+; c",                "command": "extension.vim_ctrl+c",
        "when": "editorTextFocus && vim.active && vim.overrideCtrlC  && !inDebugRepl" },
{ "key": "ctrl+; d",                "command": "extension.vim_ctrl+d",
        "when": "editorTextFocus && vim.active && !inDebugRepl" },
{ "key": "ctrl+; d",                "command": "list.focusPageDown",
        "when": "listFocus && !inputFocus" },
{ "key": "ctrl+; e",                "command": "extension.vim_ctrl+e",
        "when": "editorTextFocus && vim.active  && !inDebugRepl" },
{ "key": "ctrl+; f",                "command": "extension.vim_ctrl+f",
        "when": "editorTextFocus && vim.active  && !inDebugRepl && vim.mode != 'Insert'" },
{ "key": "ctrl+; g",                "command": "extension.vim_ctrl+g",
        "when": "editorTextFocus && vim.active  && !inDebugRepl" },
{ "key": "ctrl+; h",                "command": "extension.vim_ctrl+h",
        "when": "editorTextFocus && vim.active  && !inDebugRepl" },
{ "key": "ctrl+; i",                "command": "extension.vim_ctrl+i",
        "when": "editorTextFocus && vim.active  && !inDebugRepl" },
{ "key": "ctrl+; j",                "command": "extension.vim_ctrl+j",
        "when": "editorTextFocus && vim.active  && !inDebugRepl" },
{ "key": "ctrl+; k",                "command": "extension.vim_ctrl+k",
        "when": "editorTextFocus && vim.active  && !inDebugRepl" },
{ "key": "ctrl+; l",                "command": "extension.vim_navigateCtrlL",
        "when": "editorTextFocus && vim.active  && !inDebugRepl" },
{ "key": "ctrl+; n",                "command": "extension.vim_ctrl+n",
        "when": "editorTextFocus && vim.active  && !inDebugRepl" },
{ "key": "ctrl+; o",                "command": "extension.vim_ctrl+o",
        "when": "editorTextFocus && vim.active  && !inDebugRepl" },
{ "key": "ctrl+; p",                "command": "extension.vim_ctrl+p",
        "when": "suggestWidgetVisible && vim.active " },
{ "key": "ctrl+; q",                "command": "extension.vim_winCtrlQ",
        "when": "editorTextFocus && vim.active  && !inDebugRepl" },
{ "key": "ctrl+; r",                "command": "extension.vim_ctrl+r",
        "when": "editorTextFocus && vim.active  && !inDebugRepl" },
{ "key": "ctrl+; t",                "command": "extension.vim_ctrl+t",
        "when": "editorTextFocus && vim.active  && !inDebugRepl" },
{ "key": "ctrl+; u",                "command": "extension.vim_ctrl+u",
        "when": "editorTextFocus && vim.active  && !inDebugRepl" },
{ "key": "ctrl+; u",                "command": "list.focusPageUp",
        "when": "listFocus && !inputFocus" },
{ "key": "ctrl+; v",                "command": "extension.vim_ctrl+v",
        "when": "editorTextFocus && vim.active  && !inDebugRepl" },
{ "key": "ctrl+; w",                "command": "extension.vim_ctrl+w",
        "when": "editorTextFocus && vim.active  && !inDebugRepl" },
{ "key": "ctrl+; x",                "command": "extension.vim_ctrl+x",
        "when": "editorTextFocus && vim.active  && !inDebugRepl" },
{ "key": "ctrl+; y",                "command": "extension.vim_ctrl+y",
        "when": "editorTextFocus && vim.active  && !inDebugRepl" },
{ "key": "ctrl+; oem_4",            "command": "extension.vim_ctrl+[",
        "when": "editorTextFocus && vim.active  && !inDebugRepl" },
{ "key": "ctrl+; oem_6",            "command": "extension.vim_ctrl+]",
        "when": "editorTextFocus && vim.active  && !inDebugRepl" },
{ "key": "ctrl+; shift+2",          "command": "extension.vim_ctrl+shift+2",
        "when": "editorTextFocus && vim.active" },
{ "key": "ctrl+; pagedown",         "command": "extension.vim_ctrl+pagedown",
        "when": "editorTextFocus && vim.active  && !inDebugRepl" },
{ "key": "ctrl+; pageup",           "command": "extension.vim_ctrl+pageup",
        "when": "editorTextFocus && vim.active && !inDebugRepl" },
        { "key": "ctrl+a",                "command": "extension.vim_ctrl+a",
                "when": "editorTextFocus && vim.active && vim.mode != 'Insert'  && !inDebugRepl" },
        { "key": "ctrl+b",                "command": "extension.vim_ctrl+b",
                "when": "editorTextFocus && vim.active && vim.mode != 'Insert'  && !inDebugRepl && vim.mode != 'Insert'" },
        { "key": "ctrl+c",                "command": "extension.vim_ctrl+c",
                "when": "editorTextFocus && vim.active && vim.mode != 'Insert' && !inDebugRepl" },
        { "key": "ctrl+d",                "command": "extension.vim_ctrl+d",
                "when": "editorTextFocus && vim.active && vim.mode != 'Insert' && !inDebugRepl" },
        { "key": "ctrl+d",                "command": "list.focusPageDown",
                "when": "listFocus && !inputFocus" },
        { "key": "ctrl+e",                "command": "extension.vim_ctrl+e",
                "when": "editorTextFocus && vim.active && vim.mode != 'Insert'  && !inDebugRepl" },
        { "key": "ctrl+f",                "command": "extension.vim_ctrl+f",
                "when": "editorTextFocus && vim.active && vim.mode != 'Insert'  && !inDebugRepl && vim.mode != 'Insert'" },
        { "key": "ctrl+g",                "command": "extension.vim_ctrl+g",
                "when": "editorTextFocus && vim.active && vim.mode != 'Insert'  && !inDebugRepl" },
        { "key": "ctrl+h",                "command": "extension.vim_ctrl+h",
                "when": "editorTextFocus && vim.active && vim.mode != 'Insert'  && !inDebugRepl" },
        { "key": "ctrl+i",                "command": "extension.vim_ctrl+i",
                "when": "editorTextFocus && vim.active && vim.mode != 'Insert'  && !inDebugRepl" },
        { "key": "ctrl+j",                "command": "extension.vim_ctrl+j",
                "when": "editorTextFocus && vim.active && vim.mode != 'Insert'  && !inDebugRepl" },
        { "key": "ctrl+k",                "command": "extension.vim_ctrl+k",
                "when": "editorTextFocus && vim.active && vim.mode != 'Insert'  && !inDebugRepl" },
        { "key": "ctrl+l",                "command": "extension.vim_navigateCtrlL",
                "when": "editorTextFocus && vim.active && vim.mode != 'Insert'  && !inDebugRepl" },
        { "key": "ctrl+n",                "command": "extension.vim_ctrl+n",
                "when": "editorTextFocus && vim.active && vim.mode != 'Insert'  && !inDebugRepl" },
        { "key": "ctrl+o",                "command": "extension.vim_ctrl+o",
                "when": "editorTextFocus && vim.active && vim.mode != 'Insert'  && !inDebugRepl" },
        { "key": "ctrl+p",                "command": "extension.vim_ctrl+p",
                "when": "suggestWidgetVisible && vim.active && vim.mode != 'Insert' " },
        { "key": "ctrl+q",                "command": "extension.vim_winCtrlQ",
                "when": "editorTextFocus && vim.active && vim.mode != 'Insert'  && !inDebugRepl" },
        { "key": "ctrl+r",                "command": "extension.vim_ctrl+r",
                "when": "editorTextFocus && vim.active && vim.mode != 'Insert'  && !inDebugRepl" },
        { "key": "ctrl+t",                "command": "extension.vim_ctrl+t",
                "when": "editorTextFocus && vim.active && vim.mode != 'Insert'  && !inDebugRepl" },
        { "key": "ctrl+u",                "command": "extension.vim_ctrl+u",
                "when": "editorTextFocus && vim.active && vim.mode != 'Insert'  && !inDebugRepl" },
        { "key": "ctrl+v",                "command": "extension.vim_ctrl+v",
                "when": "editorTextFocus && vim.active && vim.mode != 'Insert'  && !inDebugRepl" },
        { "key": "ctrl+w",                "command": "extension.vim_ctrl+w",
                "when": "editorTextFocus && vim.active && vim.mode != 'Insert'  && !inDebugRepl" },
        { "key": "ctrl+x",                "command": "extension.vim_ctrl+x",
                "when": "editorTextFocus && vim.active && vim.mode != 'Insert'  && !inDebugRepl" },
        { "key": "ctrl+y",                "command": "extension.vim_ctrl+y",
                "when": "editorTextFocus && vim.active && vim.mode != 'Insert'  && !inDebugRepl" },
        { "key": "ctrl+oem_4",            "command": "extension.vim_ctrl+[",
                "when": "editorTextFocus && vim.active && vim.mode != 'Insert'  && !inDebugRepl" },
        { "key": "ctrl+oem_6",            "command": "extension.vim_ctrl+]",
                "when": "editorTextFocus && vim.active && vim.mode != 'Insert'  && !inDebugRepl" },
        { "key": "ctrl+shift+2",          "command": "extension.vim_ctrl+shift+2",
                "when": "editorTextFocus && vim.active && vim.mode != 'Insert'" },
        { "key": "ctrl+pagedown",         "command": "extension.vim_ctrl+pagedown",
                "when": "editorTextFocus && vim.active && vim.mode != 'Insert'  && !inDebugRepl" },
        { "key": "ctrl+pageup",           "command": "extension.vim_ctrl+pageup",
                "when": "editorTextFocus && vim.active && vim.mode != 'Insert' && !inDebugRepl"
        },
        {
                "key": "ctrl+oem_3",
                "command": "-workbench.action.terminal.toggleTerminal"
        },
        {
                "key": "ctrl+oem_3",
                "command": "workbench.action.terminal.focus"
        },
]
```

## c++

- [aether](https://github.com/hadeaninc/aether/blob/develop/EDITORS.md#vscode)

### vscode-ccls

- outline, call-hierarchy, member-hierarchy - very useful views

## wsl

- make sure to run code.sh from windows path for starting windows wsl integration or linux path with windows path removed to guarantee linux startup



