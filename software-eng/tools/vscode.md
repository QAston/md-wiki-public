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

## c++

- [aether](https://github.com/hadeaninc/aether/blob/develop/EDITORS.md#vscode)

### vscode-ccls

- outline, call-hierarchy, member-hierarchy - very useful views
- 



