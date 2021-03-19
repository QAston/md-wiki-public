## neovim

### usage

- `:!ls` - run shell command
- `:terminal` runs an embedded terminal emulator (which somehow works lol)

### windows setup

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
call plug#end()

if $SHELL =~ ".*pwsh.exe$"
    set shell=pwsh
    set shellquote= shellpipe=\| shellxquote=
    set shellcmdflag=-NoLogo\ -NoProfile\ -ExecutionPolicy\ RemoteSigned\ -Command
    set shellredir=\|\ Out-File\ -Encoding\ UTF8
elseif $SHELL =~ ".*powershell.exe$"
    set shell=powershell
    set shellquote= shellpipe=\| shellxquote=
    set shellcmdflag=-NoLogo\ -NoProfile\ -ExecutionPolicy\ RemoteSigned\ -Command
    set shellredir=\|\ Out-File\ -Encoding\ UTF8
elseif $SHELL =~ ".*bash$"
    set shell=bash
    set shellcmdflag=-c
else
    set shell=cmd.exe
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

## todo:
- leader key
- switching windows
- finish vimtutor

### usage

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

## build instructions

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