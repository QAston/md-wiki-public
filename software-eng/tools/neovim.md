
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