## msys2

### environment setup

1. add entries to windows terminal settings <https://www.msys2.org/docs/terminals/>
```
            {
                "guid": "{17da3cac-b318-431e-8a3e-7fcdefe6d114}",
                "name": "MINGW64 / MSYS2",
                "commandline": "C:/portable/msys/msys2_shell.cmd -defterm -here -no-start -mingw64",
                "startingDirectory": "C:/portable/msys/home/%USERNAME%",
                "icon": "C:/portable/msys/mingw64.ico"
              },
              {
                "guid": "{2d51fdc4-a03b-4efe-81bc-722b7f6f3820}",
                "name": "MINGW32 / MSYS2",
                "commandline": "C:/portable/msys/msys2_shell.cmd -defterm -here -no-start -mingw32",
                "startingDirectory": "C:/portable/msys/home/%USERNAME%",
                "icon": "C:/portable/msys/mingw32.ico"
              },
              {
                "guid": "{71160544-14d8-4194-af25-d05feeac7233}",
                "name": "MSYS / MSYS2",
                "commandline": "C:/portable/msys/msys2_shell.cmd -defterm -here -no-start -msys",
                "startingDirectory": "C:/portable/msys/home/%USERNAME%",
                "icon": "C:/portable/msys/msys2.ico"
              }
```
2. add PATH entries to /etc/profile
```
# add this to the path list
MSYS2_PATH=..."/c/portable/utils/bash"...

```
3. add entries to /etc/fstab
```
c:/Program\040Files\040(x86) /progsx86 ntfs binary,posix=0,noacl,user 0 0
c:/Program\040Files /progs ntfs binary,posix=0,noacl,user 0 0
c:/Users/QAston		/home/qasto/winhome ntfs binary,posix=0,noacl,user 0 0
```
4. update .bashrc: TODO
5. install git: TODO
set home dir: TODO


### config options

msys2.cmd flags
```
[options] [login shell parameters]
#options:
-defterm ^| -mintty ^| -conemu # defterm is the standard cmd terminal
-[use-]full-path # set MSYS2_PATH_TYPE=inherit (append win PATH), default is minimal
-mingw32 ^| -mingw64 ^| -msys[2] # shell type, what to set for clang?
-here ^| -where WORKDIR # what workdir to use
-shell # login shell
-no-start # if set, will reuse the same console host, for example ./msys2_shell.cmd -no-start -defterm will run in the same conhost
```

env variables:
```
MSYSTEM=#"MINGW32","MINGW64","MSYS", "CLANG32", "CLANG64" (clangs will appear as clang64)
MSYS2_PATH_TYPE=#minimal (default; adds standard windows paths, msys paths take priority), inherit (copies path from windows, msys paths take priority), strict (only msys paths)
```

### installing packages from source

for mingw-w32
```
cd ~/pkgbuilds

export MINGW_INSTALLS=mingw32

makepkg-mingw -sLf

pacman -u <plik wygenerowanej paczki>
```

### file paths

- [msys2 path guide](https://www.msys2.org/docs/filesystem-paths/)
- disable path conversions in MSYS1 (and git?), doesn't work in msys2: `export MSYS_NO_PATHCONV=1` 
