# bash

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