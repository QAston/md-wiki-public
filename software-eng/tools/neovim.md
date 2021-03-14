## neovim

### usage

- `:!ls` - run shell command
    - doesn't work in msys/mingw, a half-working workaround is running `alias nvim="SHELL=cmd.exe nvim"`
    - probably needs an msys-specific build: https://github.com/neovim/neovim/wiki/Building-Neovim#windows--msys2mingw
        - the build doesn't fix the issue, looks like some msys-specific patches need to be applied
    - <https://stackoverflow.com/questions/13911881/setting-cygwin-shell-path-on-vim-for-windows>
    - msys2 vim works in bash - `alias nvim vim`?
- `:terminal` runs an embedded terminal emulator (which so9mehow works lol)

### windows setup

- unzip nvim-win64.zip to portable
- the package has some gnu binutils: tee, cat, diff, tidy, xdd, so don't add it to path
- sendto shim-windows: nvim.exe, nvim-qt.exe, wind32yank.exe

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