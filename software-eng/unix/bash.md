# bash

### usage

* source [script] - start a script in a current context, all evals will be set to it 
* [bash source cheatsheet](https://mywiki.wooledge.org/BashSheet)
* [bash shortcuts cheatsheet](https://readline.kablamo.org/emacs.html)
* [bash vi mode shortcuts cheatsheet](https://readline.kablamo.org/vi.html)
* [awesome bash](https://github.com/awesome-lists/awesome-bash)
* [bash cheatsheet](https://github.com/LeCoupa/awesome-cheatsheets/blob/master/languages/bash.sh)
* <https://github.com/jlevy/the-art-of-command-line>
* <https://mywiki.wooledge.org/BashWeaknesses>
* <https://github.com/nojhan/liquidprompt>
* interactive vs login shell
    * login (interactive or not) runs /etc/profile, then first readable of ~/.bash_profile, ~/.bash_login, and ~/.profile
        * often ~/.bash_profile imports ~/.bashrc if shell is interactive
    * interactive non-login shell runs ~/.bashrc on startup
    * noninteractive shells run contents of $BASH_ENV on startup
* [writing more reliable scripts](http://redsymbol.net/articles/unofficial-bash-strict-mode/)
* [bash pitfals](https://mywiki.wooledge.org/BashPitfalls)
* <https://effective-shell.com/docs/part-5-building-your-toolkit/managing-remote-git-repositories/>
* [redirections](https://catonmat.net/bash-one-liners-explained-part-three)

### setup for linux

 * Docs: <https://wiki.archlinux.org/index.php/Bash#Tips_and_tricks> (see also related articles)
 * setup readline by copying .initrc from wslconfig repo
 * Set up bash completion:
    * pacman -S bash-completion
    * `source /usr/share/bash-completion/bash_completion` in .bashrc
 * bash config
```
# Whenever displaying the prompt, write the previous line to disk
export PROMPT_COMMAND="history -a"
# history file management
export HISTIGNORE=exit
export HISTCONTROL=ignorespace:ignoredups
export HISTSIZE=10000
export HISTFILESIZE=100000
shopt -s histappend

# history substitution - !
shopt -s histreedit # allow editing ! subs
shopt -s histverify # confirm ! subs

# disable CTRL-S terminal halt
stty -ixon
stty -ixoff
stty -ixany
```
 * set up aliases
```
# aliases
alias mk="make -f Makefile-gitignore"
alias cg='cd $(it rev-parse --show-toplevel)'

# Default to human readable figures
alias df='df -h'
alias du='du -h'
alias ll="ls -lhA"

#
# Misc :)
# alias less='less -r'                          # raw control characters
# alias whence='type -a'                        # where, of a sort
alias grep='grep --color'                     # show differences in colour
alias egrep='egrep --color=auto'              # show differences in colour
alias fgrep='fgrep --color=auto'              # show differences in colour
#
# Some shortcuts for different directory listings
alias ls='ls -hF --color=tty'                 # classify files in colour
# alias dir='ls --color=auto --format=vertical'
# alias vdir='ls --color=auto --format=long'
alias ll='ls -l'                              # long list
# alias la='ls -A'                              # all but . and ..
# alias l='ls -CF'
```
 * set up prompt
```
eval "$(~/wslconfig/ohmyposh/oh-my-posh-wsl --init --shell bash --config ~/wslconfig/ohmyposh/custom.omp.json)"
```