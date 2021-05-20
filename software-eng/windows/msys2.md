## msys2

Guide on how to configure msys2. Integration with other environments is documented in [devenv](../devenv.md).

### environment setup

#### set home dir to windows dir, edit /etc/nsswitch.conf
```
db_home: windows cygwin db
```
move files from old home dir to the windows user dir

#### add entries to windows terminal settings <https://www.msys2.org/docs/terminals/>
```
            {
                "guid": "{17da3cac-b318-431e-8a3e-7fcdefe6d114}",
                "name": "MINGW64 / MSYS2",
                "commandline": "C:/portable/msys/msys2_shell.cmd -defterm -here -no-start -mingw64",
                "icon": "C:/portable/msys/mingw64.ico"
              },
              {
                "guid": "{71160544-14d8-4194-af25-d05feeac7233}",
                "name": "MSYS / MSYS2",
                "commandline": "C:/portable/msys/msys2_shell.cmd -defterm -here -no-start -msys",
                "icon": "C:/portable/msys/msys2.ico"
              }
```

#### set up PATH

add the following to .bash_profile
```
function env_windows_path() {
  WINPATH=`/c/Windows/System32/WindowsPowerShell/v1.0/powershell -NoProfile -Command "(Get-ItemProperty -Path 'Registry::HKEY_CURRENT_USER\\Environment').Path + ';' + (Get-ItemProperty -Path 'Registry::HKEY_LOCAL_MACHINE\\SYSTEM\\CurrentControlSet\\Control\\Session Manager\\Environment').Path"`
  PATH="$PATH:$(cygpath -up "$WINPATH")"
}

case "${MSYSTEM}" in
MINGW*|CLANG*)
  if ! [ -z "$PATH_APPEND_MINGW" ]; then 
  PATH="$PATH:$(/usr/bin/cygpath -up "$PATH_APPEND_MINGW")"
  fi
  ;;
MSYS)
  if ! [ -z "$PATH_APPEND_MSYS" ]; then 
  PATH="$PATH:$(/usr/bin/cygpath -up "$PATH_APPEND_MSYS")"
  fi
  if ! [ -z "$PATH_APPEND_POSIX_COMMON" ]; then 
  PATH="$PATH:$(/usr/bin/cygpath -up "$PATH_APPEND_POSIX_COMMON")"
  fi
  ;;
esac

if ! [ -z "$PATH_APPEND_MSYS_COMMON" ]; then 
PATH="$PATH:$(/usr/bin/cygpath -up "$PATH_APPEND_MSYS_COMMON")"
fi

if ! [ -z "$PATH_APPEND_BASH_COMMON" ]; then 
PATH="$PATH:$(/usr/bin/cygpath -up "$PATH_APPEND_BASH_COMMON")"
fi
```

#### add entries to /etc/fstab
```
c:/Program\040Files\040(x86) /progsx86 ntfs binary,posix=0,noacl,user 0 0
c:/Program\040Files /progs ntfs binary,posix=0,noacl,user 0 0
#/usr/bin /bin ntfs binary,posix=0,noacl,user 0 0
```

#### cleanup pacman.comf

- remove mingw 32 bit repository

#### configure .bashrc:
```
# history file management
export HISTIGNORE=exit
export HISTCONTROL=ignorespace:ignoredups
export HISTSIZE=10000
export HISTFILESIZE=100000
export PROMPT_COMMAND="history -a" # commit history on prompt because windows doesn't let bash commit history on exit; history from other shells still won't be picked unless reloaded

shopt -s histappend

# disable CTRL-S terminal halt
stty -ixon
stty -ixoff
stty -ixany
#
#stty hup - send hup signal on disconnect

# history substitution - !
shopt -s histreedit # allow editing ! subs
shopt -s histverify # confirm ! subs

alias cg='cd $(it rev-parse --show-toplevel)'
alias ll="ls -lhA"
alias mk="make -f Makefile-gitignore"
alias df='df -h'
alias du='du -h'
alias ls='ls -hF --color=tty'
alias grep='grep --color'                     # show differences in colour
alias egrep='egrep --color=auto'              # show differences in colour
alias fgrep='fgrep --color=auto'
```

#### configure .inputrc

```
# settings

# Don't ring bell on completion
set bell-style none

# or, don't beep at me - show me
#set bell-style visible

# Expand homedir name
#set expand-tilde on

# completion

# Filename completion/expansion
set completion-ignore-case on
# Append "/" to all dirnames
set mark-directories on
set mark-symlinked-directories on
# visible-stats
# Append a mark according to the file type in a listing
set visible-stats off
set mark-directories on
#set match-hidden-files on
# immediately show all completions when no partial completions available for this word (instead of waiting for double-t>set show-all-if-unmodified on
# immediately show all completions when a completion is ambiguous for this word (instead of waiting for double-tab)
set show-all-if-ambiguous on

# history
# preserve cursor when changing history entries
set history-preserve-point on
# add * to prompt when changing history entries
set mark-modified-lines on

# 'Magic Space'
# Insert a space character then performs
# a history expansion in the line
#Space: magic-space

# show if in vi mode
set show-mode-in-prompt on

$if mode=emacs
  # custom - whitespace important - can't have whitespace before colon!
  "\C-p": history-search-backward
  "\C-n": history-search-forward
  "\e[3;5~": kill-word   # ctrl - del

  # standard msys stuff
  #"\e[2~": paste-from-clipboard      # "Ins. Key"
  #"\e[5~": beginning-of-history      # Page up
  #"\e[6~": end-of-history            # Page down

  "\e[1~": beginning-of-line      # Home Key
  "\e[4~": end-of-line            # End Key
  "\e[3~": delete-char      # Delete Key
  "\e[1;5C": forward-word   # ctrl + right
  "\e[1;5D": backward-word  # ctrl + left 
  "\e[17~": "Function Key 6"
  "\e[18~": "Function Key 7"
  "\e[19~": "Function Key 8"
  "\e[20~": "Function Key 9"
  "\e[21~": "Function Key 10"
  "\e[23~": "Function Key 11"

  $if clink
    "\x1b[27;5;32~":    clink-popup-complete # ctrl+space
    "\C-?": backward-kill-word         # Ctrl-BackSpace - works for clink, doesn't for msys
     # intercepted and handled by conhost terminal   clink-scroll-line-up # ctrl up - scroll up
     # intercepted and handled by conhost terminal   clink-scroll-line-down# ctrl down - scroll down
    "\e[5;5~": clink-scroll-page-up # ctrl pgup - scroll page up
    "\e[6;5~": clink-scroll-page-down # ctrl pgdown - scroll page down
    "\e[1;5H": clink-scroll-top # ctrl home - scroll beginning
    "\e[1;5F": clink-scroll-bottom # ctrl end - scroll end
  $else
    "\C-H": backward-kill-word # ctrl - backspace, doesn't work for clink
  $endif
$endif
```

#### configure prompt

* download <https://github.com/JanDeDobbeleer/oh-my-posh/releases/download/v3.96.0/posh-windows-wsl-amd64.7z> if not in portable/oh-my-posh
* add to .bashrc
```
export POSH_THEME=C:/portable/oh-my-posh/custom.omp.json

function _update_ps1() {
    PS1="$(C:/portable/oh-my-posh/oh-my-posh.exe --pwd \\w --config $POSH_THEME --error $?)"
}

if [ "$TERM" != "linux" ] && [ -x "$(command -v C:/portable/oh-my-posh/oh-my-posh.exe)" ]; then
    PROMPT_COMMAND="_update_ps1; $PROMPT_COMMAND"
fi
```

#### update after installing

```
# in msys shell
pacman -Syu
# if msys closes after the update to delete execs, finish the update running in msys shell
pacman -Su
```

#### install useful packages
```
pacman -S --needed base-devel binutils mingw-w64-x86_64-toolchain cmake mingw64/mingw-w64-x86_64-diffutils ncurses rlwrap
```
#### set up ABS and AUR

ABS:
```
mkdir ~/abs
cd ~/abs
git clone git@github.com:msys2/MSYS2-packages.git
```

#### Install git 

1. follow <https://github.com/git-for-windows/git/wiki/Install-or-update-inside-MSYS2,-Cygwin-or-Git-for-windows-itself>
- in mingw64: curl https://raw.githubusercontent.com/git-for-windows/build-extra/HEAD/git-extra/getgit | bash
- will add git.exe shim to c:/portable/msys/cmd
- sadly, will also add a bunch of binaries to /mingw64, you'll need to install stuff with --override "*" to override these later
2. add git to msys2 (by default it's only in mingw64)
- install a version of git from MSYS-packages, but modify to not install binaries, as those will be used from git-for-windows
```
diff --git a/git/PKGBUILD b/git/PKGBUILD
index 0db0a091..a5ed382e 100644
--- a/git/PKGBUILD
+++ b/git/PKGBUILD
@@ -68,7 +68,7 @@ prepare() {
   patch -p1 -i "${srcdir}"/git-1.9.0-manifest-msys2.patch
   patch -p2 -i "${srcdir}"/git-2.3.5-mingw-pwd.patch
   patch -p1 -i "${srcdir}"/git-2.8.2-Cygwin-Allow-DOS-paths.patch
-  patch -p1 -i "${srcdir}"/git-tcsh-completion-fixes.patch
+  #patch -p1 -i "${srcdir}"/git-tcsh-completion-fixes.patch

   local _arch=
   if [ "${CARCH}" == 'x86_64' ]; then
@@ -110,18 +110,18 @@ check() {
   # We used to use this, but silly git regressions:
   #GIT_TEST_OPTS="--root=/dev/shm/" \
   # https://comments.gmane.org/gmane.comp.version-control.git/202020
-  make \
-    NO_SVN_TESTS=y \
-    DEFAULT_TEST_TARGET=prove \
-    GIT_PROVE_OPTS="$jobs -Q" \
-    GIT_TEST_OPTS="--root=/tmp/git-test" \
-    test
+  #make \
+  #  NO_SVN_TESTS=y \
+  #  DEFAULT_TEST_TARGET=prove \
+  #  GIT_PROVE_OPTS="$jobs -Q" \
+  #  GIT_TEST_OPTS="--root=/tmp/git-test" \
+  #  test
 }

 package() {
   export PYTHON_PATH='/usr/bin/python'
   cd "${srcdir}/${pkgname}-${pkgver}"
-  make INSTALLDIRS=vendor DESTDIR="$pkgdir" install
+  #make INSTALLDIRS=vendor DESTDIR="$pkgdir" install
   make INSTALLDIRS=vendor DESTDIR="${pkgdir}" install-man
   #make INSTALLDIRS=vendor DESTDIR="${pkgdir}" install-info

@@ -139,9 +139,9 @@ package() {
   cp -a ./contrib/* ${pkgdir}/usr/share/git/

   # scripts are for python
-  sed -i 's|#![ ]*/usr/bin/env python$|#!/usr/bin/python|' \
-    $(find "${pkgdir}" -name '*.py') \
-    "${pkgdir}"/usr/lib/git-core/git-p4 \
-    "${pkgdir}"/usr/share/git/remote-helpers/git-remote-bzr \
-    "${pkgdir}"/usr/share/git/remote-helpers/git-remote-hg
+  #sed -i 's|#![ ]*/usr/bin/env python$|#!/usr/bin/python|' \
+  #  $(find "${pkgdir}" -name '*.py') \
+  #  "${pkgdir}"/usr/lib/git-core/git-p4 \
+  #  "${pkgdir}"/usr/share/git/remote-helpers/git-remote-bzr \
+  #  "${pkgdir}"/usr/share/git/remote-helpers/git-remote-hg
 }
```
- build and install
```
cd ~/abs/git
makepkg -s
pacman -U <git>
pacman -S git-extras # supporting modules
```
- disable updates of the git package
```
# in /etc/pacman.conf
IgnorePkg   =git
```
3. Configure git by following setup in [git](../tools/GIT.md)
4. install git command extensions:
  - clone https://github.com/newren/git-filter-repo
  - copy binaries (.py and executable linkg) to c:\portable\msys2\cmd

#### install win-sudo (bash-only)

1. clone <https://github.com/imachug/win-sudo>
2. symlink sudo, su, sudobackend and sudorc to c:/portable/bin/msys_common
3. looks like output redirection doesn't always work, you can add sleep to fix that:
```
sudo sleep 3 && echo "works"
```
4. could run it from bash/powershell using bash-mingw64-login?

#### make bash scripts and mingw64/msys commands convenient to use from other shells

- bash-msys and bash-mingw64 are added as shims in devenv.md guide
- other executables can also be shimmed as needed
- for packages in mingw64 but not available in msys you can add a shim if needed too
- for packages which are both in msys and mingw it's usually better to install both versions, but you can also do what was done for git

### guides

- [msys2 wiki](https://www.msys2.org/wiki/Home/)
- [building without package manager](http://www.gaia-gis.it/gaia-sins/mingw64_how_to.html)
- [mingw setup for llvm](https://github.com/valtron/llvm-stuff/wiki/Set-up-Windows-dev-environment-with-MSYS2)
- [mingw setup for openfgrameworks](https://openframeworks.cc/setup/msys2/)
- [msys2 path guide](https://www.msys2.org/docs/filesystem-paths/)
  - this means that `cmd.exe /c` will by default be converted to `cmd.exe C:\`, unless escaped `cmd.exe //c` or disabled using `MSYS2_ARG_CONV_EXCL="*"`
  - this is also true for env variables: MSYS2_ENV_CONV_EXCL="*" (which will sadly prevent PATH from being fixed)
  - the conversion is only applied when msys apps execute native apps, not the other way around
- [msys here](https://gist.github.com/elieux/ef044468d067d68040c7), windowsterminal-shell is much better though

### useful utilities

- filesystem package:
  - `shell`
    - `shell [path]` opens windows explorer if path exists and isn't executable, if file explorer will open the default handler for the file
    - `shell [executable file or unknown]` run a command 
    - `source shell msys2|mingw32|mingw64` switches shell type by reimporting profile
  - `dep-search`
  - `cmd` run args in cmd
  - `start` runs `cmd /c start`
- /dev/clipboard
  - prints contents of clipboard
- regtool - editing registry
- cygpath - path conversion
- pacman
    - pacman -Qo find package owning a file (must specify extension otherwise won't find anything)

### start options

msys2_shell.cmd flags
```
[options] [login shell parameters]
#options:
-defterm ^| -mintty ^| -conemu # defterm is the standard cmd terminal
-[use-]full-path # set MSYS2_PATH_TYPE=inherit (append win PATH), default is minimal
-mingw32 ^| -mingw64 ^| -msys[2] # shell type, what to set for clang?
-here ^| -where WORKDIR # what workdir to use, sets CHERE_INVOKING in both cases
-shell # login shell
-no-start # if set, will reuse the same temrinal/console host, for example ./msys2_shell.cmd -no-start -defterm will run in the same conhost
```

env variables:
```
MSYSTEM=#"MINGW32","MINGW64","MSYS", "CLANG32", "CLANG64" (clangs will appear as clang64)
MSYS2_PATH_TYPE=#minimal (default; adds standard windows paths, msys paths take priority), inherit (copies path from windows, msys paths take priority), strict (only msys paths)
CHERE_INVOKING=0#ignore workdir 1 - start in the workdir
MSYS=#msys dll flags (space separated), documented here https://cygwin.com/cygwin-ug-net/using-cygwinenv.html
```

there are also mintty launchers which launch the shell with the config values set in their ini files

### overview

- [overview](https://www.msys2.org/wiki/MSYS2-introduction/)
- posix/msys shell
  - use for running pacman (and other posix/msys software that could be broken by mingw64 binaries overriding msys ones), makepkg (the msys version), and building for posix/msys without makepkg, nothing else
    - pacman [depends](https://github.com/msys2/MSYS2-packages/blob/master/pacman/PKGBUILD#L13) on posix/msys sofware, and so do pre/post-install scripts inside packages
    - if the mingw64 version behaves differently (not handling posix paths, having different flags/implementations) these would break in the mingw64 shell
  - contains packages that are needed to build software for mingw shells and a few selected packages which are way too difficult to port to mingw
  - msys shell is 32 or 64 bit depending on the installer
- mingw32/64 shells
  - use for building packages for windows with or without makepkg-mingw and for everything else
    - (makepkg-mingw sets the env for each build target so it can be run from any shel including msys)
  - used for running the windows software in an environment that can also use msys software like bash as a fallback, mingw32/64 software takes priority
- mingw32/64 software
  - purely native software which can be used outside of mingw32/64/msys shells, includes gcc toolchains and binutils
  - ideally there would be a way to alias/shim/other-way-to-separate for selected software to be able to add selected things to default windows PATH without exporting everything
  - packages are in separate package repo (mingw-w64)
  - packages have a name prefix (mingw-w64-x86_64-) to not collide with msys packages


### packages

- [online packages list](https://packages.msys2.org/search)
- [package management](https://www.msys2.org/docs/package-management/)
- [making and building packages](https://www.msys2.org/wiki/Creating-Packages/)
- [package manager command list](https://wiki.archlinux.org/index.php/Pacman/Rosetta)
- looking at aur/arch packages can give a good starting point for adding a new package
- [tips on porting to windows](https://www.msys2.org/wiki/Porting/)

#### installing packages from pkgbuild

for mingw-w64 only
```
cd ~/pkgbuilds

export MINGW_INSTALLS=mingw64

makepkg-mingw -sLf

pacman -u <file>
```
