# GIT

## setup

* windows only - set up crlf handling to cooperate well with linux users
```
git config --global core.autocrlf false

touch ~/.gitattributes_global
git config --global core.attributesFile ~/.gitattributes_global
# convert most files to core.eol line ending, with platform specific files having their endings preserved:
wget -O - https://raw.githubusercontent.com/alexkaratarakis/gitattributes/master/{Common,C++,VisualStudio,Java,CSharp}.gitattributes > ~/.gitattributes_global
```
* set up always ignored files:
```
touch ~/.gitignore_global
echo '*-gitignore' > ~/.gitignore_global
echo '*-gitignore.*' >> ~/.gitignore_global
# gitignore global
git config --global core.excludesfile ~/.gitignore_global
```
* other config
```
# config
git config --global mergetool.keepBackup false
git config --global push.default simple
# aliases
git config --global alias.st status
git config --global alias.unstage 'reset HEAD --'
git config --global alias.lg "log --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr %an)%Creset' --abbrev-commit --date=relative"
```

## usage

- gitignore: files without / are ignored on every directory level
- local/private gitignore file .git/info/exclude

## commands

* `git update-index --chmod=+x <your_file>` make executable
* `git config submodule.recurse true` download submodules
* `git grep ` searches in history
* `git config credential.helper store` stop confirming pass
* `git update-index \--[no-]assume-unchanged] [target]` - stop tracking file
* `git add --renormalize .` - normalize line endings of text files in the repository

## links

* [visual guide to commands](http://marklodato.github.io/visual-git-guide/index-en.html)
* [progit.org/book/](http://progit.org/book/)
* [Effectively Using Git With Subversion | Viget Extend](http://www.viget.com/extend/effectively-using-git-with-subversion/)
* [How To Use Git-SVN as the Only Subversion Client You’ll Need at Everything In Between](http://maymay.net/blog/2009/02/24/how-to-use-git-svn-as-the-only-subversion-client-youll-need/)
* [git-svn-tutorial – Parrot](http://trac.parrot.org/parrot/wiki/git-svn-tutorial)
* [Aliases in Git I couldn’t be without | Gaui.is](http://gaui.is/aliases-in-git-i-couldnt-be-without/)
* [Version Control with Subversion](http://svnbook.red-bean.com/)
* [history - error combining git repositories into subdirs - Stack Overflow](http://stackoverflow.com/questions/7798142/error-combining-git-repositories-into-subdirs)
* [Git Commands and Best Practices Cheat Sheet | zeroturnaround.com](http://zeroturnaround.com/rebellabs/git-commands-and-best-practices-cheat-sheet/)