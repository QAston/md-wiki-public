## neovim

### setup windows

- get at least version 0.5x because it has lsp support
- unzip nvim-win64.zip to portable
- the package has some gnu binutils: tee, cat, diff, tidy, xdd, so don't add it to path
- sendto shim-windows: nvim.exe, nvim-qt.exe, wind32yank.exe
- install vimplug
```powershell
# run in powershell
iwr -useb https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim |`
    ni "$(@($env:XDG_DATA_HOME, $env:LOCALAPPDATA)[$null -eq $env:XDG_DATA_HOME])/nvim-data/site/autoload/plug.vim" -Force
```
- add config `$env:LOCALAPPDATA/nvim/init.vim`
```
#New-Item -Force $env:LOCALAPPDATA/nvim/init.vim
call plug#begin()
Plug 'junegunn/vim-plug'
Plug 'unblevable/quick-scope'
if !exists('g:vscode')
    Plug 'editorconfig/editorconfig-vim'
endif
call plug#end()

" save the backup and swap files in a single directory
let g:backupdir=expand(stdpath('data') . '/backup')
if !isdirectory(g:backupdir)
  execute '!mkdir '. g:backupdir
endif
let g:backupdir=g:backupdir . '//'
let &backupdir=g:backupdir
let &directory=g:backupdir

if has('win32')
if $SHELL =~ ".*pwsh.exe$"
    set shell=pwsh
    set shellcmdflag=-Command
    set shellquote= shellpipe=\| shellxquote=
    set shellcmdflag=-NoLogo\ -NoProfile\ -ExecutionPolicy\ RemoteSigned\ -Command
    set shellredir=\|\ Out-File\ -Encoding\ UTF8
elseif $SHELL =~ ".*powershell.exe$"
    set shell=powershell
    set shellcmdflag=-Command
    set shellquote= shellpipe=\| shellxquote=
    set shellcmdflag=-NoLogo\ -NoProfile\ -ExecutionPolicy\ RemoteSigned\ -Command
    set shellredir=\|\ Out-File\ -Encoding\ UTF8
elseif $SHELL =~ ".*bash$"
    set shell=c:\portable\msys\usr\bin\bash.exe
    set shellcmdflag=wincommand " -c by default; here replaced with a script that checks if first arg is a bat file to execute it in cmd
    set shellxquote=
    set shellxescape=
    set shellquote=
    set shellpipe=2>&1\|\ tee
    set shellredir=>%s\ 2>&1
    set shellslash
else
    set shell=cmd.exe
endif
endif

" unmap space for normal, visual and select and operator
noremap <SPACE> <Nop>
" set space as default leader
let mapleader=" "

" enable quickscope
let g:qs_highlight_on_keys = ['f', 'F', 't', 'T']

" show keys in status line
set showcmd

" vscode specific stuff
if exists('g:vscode')
" start in insert mode
    au BufEnter * start
    highlight QuickScopePrimary guifg='#afff5f' gui=underline ctermfg=155 cterm=underline
    highlight QuickScopeSecondary guifg='#5fffff' gui=underline ctermfg=81 cterm=underline
else
    set relativenumber
    set number
    highlight QuickScopePrimary guifg='#afff5f' gui=underline ctermfg=155 cterm=underline
    highlight QuickScopeSecondary guifg='#5fffff' gui=underline ctermfg=81 cterm=underline
endif
```
- install plugins
```
# in vim
:PlugInstall
```
- set up windows env variables:
    - EDITOR=nvim
    - VISUAL=nvim

### setup wsl2

```
pacaur -S neovim-nightly-bin
mkdir ~/.config/nvim/
sh -c 'curl -fLo "${XDG_DATA_HOME:-$HOME/.local/share}"/nvim/site/autoload/plug vim --create-dirs \
       https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim'

# sync windows and linux config
cp "/mnt/c/Users/qasto/AppData/Local/nvim/init.vim" ~/.config/nvim/init.vim
dos2unix ~/.config/nvim/init.vim
```


### usage

- `:!ls` - run shell command
- `:terminal` runs an embedded terminal emulator (which somehow works lol)
- `:edit $MYVIMRC`

#### concepts

- shada - shared data - a file which stores registers and command listory shared between all the sessions
- buffer - in memory text of a file, filename displayed at the bottom
    - can be active (shown in a window), hidden(loaded but not shown) or inactive(unloaded and not shown)
- window - a viewport on a buffer, can have multiple windows on a single buffer, or multiple windows pointing to different buffers
    - split windows are windows displayed side by side at the same time
- tab - collection of windows, name displayed at the top of the screen
- registers - store deleted and yanked (copied) text
    - read only registers:
        - ". - last inserted text
        - "% - last current file path
        - ": - most recent command
        - "# - alternate file (file you switch to with ctrl-^)
        - "0 - last yank
        - "" - unnamed register - last delete or yank
        - "/ - last search pattern
        - "- - last small (less than a line) delete
    - special registers
        - "= - expression register - pasting from it will prompt you to enter vimscript expression
        - "_ - black hole register - use to throw away
        - "+ and "* - system clipboard registers
            - equivalent on windows, on linux * is selection buffer (last selection, aka PRIMARY) and + is the real clipboard buffer
            - use `set clipboard+=unnamedplus` to make + the default register for all ops which have optional register arg
            - neovim uses "clipboard privders" for these, by default `xclip` on linux when `$DISPLAY` is set, which only works if windows x server is running

        
#### config files

- by default nvim stores config files in `~/AppData/Local/nvim` and user data in `~/AppData/Local/nvim-data` (see :h standard-path)
    - startup file is `~/AppData/Local/nvim/init.vim` or init.lua (not both at the same time)
    - $VIM is implicitly set to `C:\portable\nvim\Neovim\share\nvim`

#### managing plugins

- add/remove between plug#begin/end
- restart vim or reload config using `source $MYVIMRC` (MYVIMRC is defined inside vim)
- `:PlugInstall` to install plugins
- `:PlugUpdate` to update
- `:PlugClean` to remove plugins no longer in plug#begin/end list
- these only work in cmd (uses batch scripts)

#### keybinds

- normal
    - window/tab management
        - `Ctrl+w T` - move current window into a tab
        - `gt` - next tab
        - `gT` - previous tab
        - `:tabs` - display status of all tabs
        - `:tabonly` - close all other tabs
        - `:tabclose` - close current tab and contained windows
        - `:tabnew <file>` - open file in new tab
        - `:tab ball` - edit all buffers as tabs
        - `:tabdo <command>` - run command in all tabs
        - `:buffers`/`:ls` - display status of buffers
        - `:e <file>` - edit a file in new buffer in the current window
        - `:bnext` - switch to next buffer
        - `:bprevious` - switch to previous
        - `:buffer <file>` - switch to buffer containing file
        - `:bdelete` - close a buffer (close a file)
        - `:split <file>` - open a file in new split window
        - `:vsplit <file>` - open a file in new vertically split window
        - `:vert ball` - open all buffers in vertical windows
        - `Ctrl+w s` - split window
        - `Ctrl+w v` - split window vertically
        - `Ctrl+w w` - switch windows
        - `Ctrl+w q` - close window
        - `Ctrl+w x` - swap window positions with the last used window
        - `Ctrl+w h` - switch to left window
        - `Ctrl+w l` - switch to right window
        - `Ctrl+w j` - switch to below window
        - `Ctrl+w k` - switch to above window
    - registers
        - `:reg` - show contents of all registers
        - `:reg <regname>` - show content of register
        - `<regname>p` - paste contents of register
        - `<regname>y` - copy into register
    - filesystem
        - `:r <file>` read contents of file and insert at cursor
        - `:w <file>` save selection to file
    - saving and exiting
        - `:w`- write (save) the file, but don't exit
        - `:w !sudo tee %` - write out the current file using sudo
        - `:wq` or `:x` or `ZZ` - write (save) and quit
        - `:q` - quit (fails if there are unsaved changes)
        - `:q!` or `ZQ` - quit and throw away unsaved changes
        - `:wqa` - write (save) and quit on all tabs
    - `K` - open help page for word under cursor 
    - motions
        - b/w/e/B/W/E - beginning of previous word/ beginning next word/ end of current word
            - CAPITAL moves WORDS (whitespace separated), while lowercase moves words(operator separated)
    - modes
        - i/a - to insert mode (before/after) cursor
        - I/A - to insert mode at (beginning/end) of the line
        - o/O - add a line (below/above) the current one and go to insert mode
        - v - visual mode
        - ctrl+v - visual block mode
        - shift+v - line visual mode
        - R - replace (overtype) mode (replacing chars with >1 size (like tab) will move things)
        - gR - visual replace mode (replacing chars with >1 size (like tab) will not move things, editor will compensate using other chars instead)
- visual
    - o - move cursor to the other end of selection or opposite corner of block selection
    - O - move cursor to the other corner of block selection
    - (shift)(ctrl)v - change visual mode type
    - using text-objects extends the selection by that object in the direction of the cursor for or if inside a block-object expands the block outwards
        - `a<object>` - includes surrounding whitespace (or brackets, or quotes)
        - `i<object>` - doesn't include surrounding whitespace(or brackets, or quotes), repeat once to include what `a` would include
        - see :help text-objects for full details
        - `<a/i>w` - a word
        - `<a/i>W` - a WORD
        - `<a/i>b` - a block with ()
        - `<a/i>B` - a block with {}
        - `<a/i>t` - a block with <> tags
        - `<a/i>s` - a sentence
        - `<a/i>p` - a paragraph
        - `<a/i>'` - a '-quoted string
        - `<a/i>"` - a "-quoted string
        - ``<a/i>` `` - a `-quoted string
        - `<a/i>[` - text between [ brackets
        - `<a/i>(` - text between ( brackets
        - `<a/i><` - text between < brackets
        - `<a/i>{` - text between { brackets
    - operators - act on selection and move back to normal mode
        `>` - shift text right
        `<` - shift text left
        `y` - yank (copy) marked text
        `d` - delete marked text
        `~` - switch case
        `u` - change marked text to lowercase
        `U` - change marked text to uppercase
        `=` - format
        `c` - change
        `!` - execute shell command on selection
        `:` - execute Ex command on the selection
    - motions select the area between the cursor and motion target
- insert
    - `ctrl+r <regname>` - paste register
    - `ctrl+a ` - paste last insert
    - `ctrl+x ctrl+<many keys>` - completion keys
        - `ctrl+f` - autocomplete/insert filename
        - `ctrl+o` - guess type of item in front of the cursor and find first match
        - `ctrl+n/p` - keyword completion (see standalone `ctrl+n/p`)
        - repeated presses of `ctrl+<many keys>` switch to next found completion of that type
    - `ctrl+n/p` - keyword completion (next/previous) - complete for keyword in front of the cursor
        - repeated presses replace the previous completion with the another completion of the same type (see ctrl+x keybinds family)
        - see :help complete-functions
        - <https://vim.fandom.com/wiki/Any_word_completion>
    - `ctrl+v` - insert raw keypress/3 digit decimal number as a byte
    - `ctrl+k <char1> <char2>` - enter a digraph 
        - `:digraphs` shows available digraphs
    - `ctrl+o` - execute single normal-mode command and return to insert mode
    - `ctrl+]` - expand abbreviation under cursor (abbreviations are defined using :abbr command)
    - other ctrl commands are useless and can be ignored
    - todo: port standard keyboard mappings to insert mode?
        - alt+left/right for jumplist
        - shift for selection instead of being dupe of ctrl
        - tab/shifttab for indent/deindent?
        - ctrl+space for completion?

#### keyboard mappings

- [free keys list](https://vim.fandom.com/wiki/Unused_keys)
- other free keys:
    - `ctrl+shift+<key>`
    - `<space>`
        - see <https://stackoverflow.com/questions/446269/can-i-use-space-as-mapleader-in-vim> for space: leader
        - <https://superuser.com/questions/693528/vim-is-there-a-downside-to-using-space-as-your-leader-key>
    - `ctrl+k` - enter digraph in insert mode (see :digraph for list), unbound in normal mode
        - rebind ctrl-k to something else because digraphs can actually be useful?
- keys which are kind of obsolete because I have good keyboard layout:
    - `x` - use delete instead
    - `hjkl` - arrows
    - shift+arrows/pgup/down/end/begin

#### feature flags

- :h feature-list lists win32, wsl and unix, but not win32unix, which is what vim uses for msys/cygwin
    - neovim doesn't have an msys/cygwin build variant
- looks like has('wsl') flag isn't really well supported: 
    - detection is compile time only <https://github.com/neovim/neovim/pull/7330>
    - and it doesn't work by default   <https://github.com/neovim/neovim/issues/12642>
- a lot of plugins assume has(win32) means cmd shell and has(unix) means linux 

#### shell support

- run with `SHELL="" nvim` to run with cmd, otherwise it'll run the parent shell

#### todo - terminal

- should have xterm set for colors? some things advise `export TERM=xterm-256color`
- <https://neovim.io/doc/user/term.html>
    - should it be set to vtpcon, xterm-256color or something else?

## configuration tips

- <https://devhints.io/vimscript>
    - <https://bugfactory.io/blog/customizing-vim-with-environment-variables/>
- <https://github.com/nanotee/nvim-lua-guide>
- <https://github.com/svermeulen/vimpeccable>
- <https://neovim.io/doc/user/lua.html>
- example configs:
    - <https://github.com/jdhao/nvim-config>
    - <https://gabrielpoca.com/2019-11-13-a-bit-more-lua-in-your-vim>
    - <https://teukka.tech/luanvim.html>
    - <https://github.com/jamestthompson3/vimConfig>
    - <https://github.com/glepnir/nvim>

## plugins

- <https://github.com/neovim/neovim/wiki/Related-projects>
- <https://github.com/junegunn/vim-plug>
    - <https://github.com/junegunn/vim-plug/wiki/tutorial>
- <https://github.com/neovim/nvim-lspconfig>
- <https://github.com/preservim/nerdtree>
- [completion engine from vscode ported to vim](https://github.com/neoclide/coc.nvim)

## build instructions

- not actually needed - prebuilt vim works just fine
- [msys](https://github.com/neovim/neovim/wiki/Building-Neovim#windows--msys2mingw)
    - list platform as win32, not win32unix
- [cygwin](https://github.com/neovim/neovim/wiki/Building-Neovim#cygwin)
    - https://github.com/cascent/neovim-cygwin/issues/5 - list platform as unix, not win32unix

### linux setup

install pacman -S neovim
 
install vim plug https://github.com/junegunn/vim-plug/wiki/tutorial

vim plug can be installed using aur

https://github.com/neovim/nvim-lsp
https://neovim.io/doc/user/lsp.html



config
``` 
call plug#begin('~/.config/nvim/plugged')
Plug 'neovim/nvim-lsp'
call plug#end()

:lua require'nvim_lsp'.ccls.setup{ cmd = {'ccls', '-v2', '--init={"compilationDatabaseDirectory":"out"}'} }
autocmd Filetype rust,python,go,c,cpp setl omnifunc=lsp#omnifunc
nnoremap <silent> gd    <cmd>lua vim.lsp.buf.declaration()<CR>
nnoremap <silent> <c-]> <cmd>lua vim.lsp.buf.definition()<CR>
nnoremap <silent> K     <cmd>lua vim.lsp.buf.hover()<CR>
nnoremap <silent> gD    <cmd>lua vim.lsp.buf.implementation()<CR>
nnoremap <silent> <c-k> <cmd>lua vim.lsp.buf.signature_help()<CR>
nnoremap <silent> 1gD   <cmd>lua vim.lsp.buf.type_definition()<CR>
nnoremap <silent> gr    <cmd>lua vim.lsp.buf.references()<CR>
```

ccls -index . -v=2 --init='{"compilationDatabaseDirectory":"out"}' shouldn't crash!

### tools

- <https://github.com/nvim-telescope/telescope.nvim>