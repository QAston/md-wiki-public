## neovim

### usage

- `:!ls` - run shell command
- `:terminal` runs an embedded terminal emulator (which somehow works lol)

### windows setup

- unzip nvim-win64.zip to portable
- the package has some gnu binutils: tee, cat, diff, tidy, xdd, so don't add it to path
- sendto shim-windows: nvim.exe, nvim-qt.exe, wind32yank.exe

#### msys/bash/cygwin support

- neovim doesn't have an msys/cygwin build variant
    - :h feature-list lists win32, wsl and unix, but not win32unix, which is what vim uses for msys/cygwin
- a half-working workaround for making shell commands work in msys/mingw is running `alias nvim="SHELL=cmd.exe nvim"`
    - <https://stackoverflow.com/questions/13911881/setting-cygwin-shell-path-on-vim-for-windows>
    - msys2 vim works in bash - `alias nvim vim`?
- some people claim wsl works? https://stackoverflow.com/questions/59804779/configure-nvim-and-oni-to-use-bash-on-wsl-in-windows
- msys build: https://github.com/neovim/neovim/wiki/Building-Neovim#windows--msys2mingw is the build aimed at win32, doesn't have bash support
    - looks like it'd need a custom patch
- there's a cygport source you can build, but it lists the platform as unix:
    - https://github.com/cascent/neovim-cygwin
    - https://github.com/cascent/neovim-cygwin/issues/5
- looks like has('wsl') flag isn't really well supported: 
    - detection is compile time only <https://github.com/neovim/neovim/pull/7330>
    - and it doesn't work by default   <https://github.com/neovim/neovim/issues/12642>

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