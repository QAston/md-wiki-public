# GIT

[visual guide to commands](http://marklodato.github.io/visual-git-guide/index-en.html)
    
```bash
#stop confirming pass
git config credential.helper store

# stop tracking file
git update-index \--[no-]assume-unchanged] [target]
```

local/private gitignore file .git/info/exclude

gitignore:

files without / are ignored on every directory level

## setup

* `git config --global mergetool.keepBackup false` disable orig files generation
* set up always ignored files:
```
touch ~/.gitignore_global
git config --global core.excludesfile ~/.gitignore_global
echo "*-gitignore" > ~/.gitignore_global
```

# commands

* `git update-index --chmod=+x <your_file>` make executable
* `git config submodule.recurse true` download submodules
* `git config --global mergetool.keepBackup false` disable orig files generation
* `git grep ` searches in history

## links

* [progit.org/book/](http://progit.org/book/)
* [Effectively Using Git With Subversion | Viget Extend](http://www.viget.com/extend/effectively-using-git-with-subversion/)
* [How To Use Git-SVN as the Only Subversion Client You’ll Need at Everything In Between](http://maymay.net/blog/2009/02/24/how-to-use-git-svn-as-the-only-subversion-client-youll-need/)
* [git-svn-tutorial – Parrot](http://trac.parrot.org/parrot/wiki/git-svn-tutorial)
* [Aliases in Git I couldn’t be without | Gaui.is](http://gaui.is/aliases-in-git-i-couldnt-be-without/)
* [Version Control with Subversion](http://svnbook.red-bean.com/)
* [history - error combining git repositories into subdirs - Stack Overflow](http://stackoverflow.com/questions/7798142/error-combining-git-repositories-into-subdirs)
* [Git Commands and Best Practices Cheat Sheet | zeroturnaround.com](http://zeroturnaround.com/rebellabs/git-commands-and-best-practices-cheat-sheet/)