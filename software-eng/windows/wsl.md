## setup windows - wsl integration

### Setup X on the windows side

1. No longer needed/possible?: Mount the linux root as a drive (go to `\\wsl$\` in explorer, right click on a linux dir and select mount as network drive)
2. Install an x server
     * Cygwin x
         * Install cygwin
         * Install xlaunch package
         * "C:\portable\cygwin\bin\xwin.exe" -ac -multiwindow -nowgl -listen tcp
     * Vcxsrv - seems to have window cleanup problems in multiwindow mode
         * Install <https://sourceforge.net/projects/vcxsrv/>
         * create a file called `multiwin.xlaunch` 
```xml
<?xml version="1.0" encoding="UTF-8"?>
<XLaunch WindowMode="MultiWindow" ClientMode="NoClient" LocalClient="False" Display="-1" LocalProgram="xcalc" RemoteProgram="xterm" RemotePassword="" PrivateKey="" RemoteHost="" RemoteUser="" XDMCPHost="" XDMCPBroadcast="False" XDMCPIndirect="False" Clipboard="True" ClipboardPrimary="False" ExtraParams="-multiwindow -ac -nowgl -reset -background none -xkblayout gb -xkbmodel kinesis" Wgl="False" DisableAC="True" XDMCPTerminate="False"/>
```
         * You can run in a single window mode to prevent the window cleanup glitch, or just restart the server
4. Run the shortcut anytime you want gui access
5. Window managers - you can use linux window manager instead of multiwindow x server
     * "xserver" -ac -nowgl -screen 0 
     * pacman -S kwin openbox
     * `openbox`/`kwin_x11`
     * add `-rootless` to run in rootless mode - don't display root window for winmgr - x windows can overlap with regular windows
6. Troubleshooting - visual glitches
     * Some apps have broken rendering code and restarting won’t help - try a different ui style/widget set instead, maybe an app update
     * Vcxsrv needs restarting when a part of the screen is glitched out (or switch to cygwin x win instead)
7. Documentation:
    * <https://x.cygwin.com/docs/man1/XWin.1.html> - flags
    * <https://x.cygwin.com/docs/man5/XWinrc.5.html> - config file
8. Set up [env variable forwarding](https://devblogs.microsoft.com/commandline/share-environment-vars-between-wsl-and-windows/):
    * set windows env var `WSLENV` to `PATH_VAR_ALL/l:PATH_APPEND_BASH_COMMON/l`


### Setup X on linux side

1. Login as a regular user
2. Modify `.bashrc` to configure x variables
     * `export __GLX_VENDOR_LIBRARY_NAME=mesa`
     * `export LIBGL_ALWAYS_INDIRECT=0`
     * `export DISPLAY=$(cat /etc/resolv.conf | grep nameserver | awk '{print $2; exit;}'):0.0`
3. relogin
4. sudo pacman -S xterm mesa-demos noto-fonts ttf-cascadia-code xorg-xrdb xclip
5. pacaur -S nerd-fonts-complete
6. Xterm should launch properly
- configure ~/.Xresources
```
xterm*faceName: Cascadia Mono
xterm*faceSize: 12
xterm*background: black
xterm*foreground: lightgray
```
- load the config `xrdb -merge ~/.Xresources`
7. Glxgears should launch properly

### Set up pulse audio (windows side)

1. get cygwin, install pulseaudio package
2. edit configuration
```
# in /etc/pulse/default.pa find load-module module-native-protocol-tcp and replace with:
load-module module-native-protocol-tcp auth-ip-acl=127.0.0.1;172.16.0.0/12
```
```
# in /etc/pulse/daemon.conf set
daemonize = yes
use-pid-file = no
exit-idle-time = -1 
```
3. add a script to start pulseaudio daemon:
```
@echo off
setlocal enableextensions
set TERM=
cd /d "C:\portable\cygwin\bin" && .\bash --login -c "HOME=/home/qasto pulseaudio"
```
4. make sure the firewall access is enabled

### Set up pulse audio (linux side)

1. install pulseaudio client
```
sudo pacman -S pulseaudio pulseaudio-alsa
```
2. set up pulseaudio connection to windows - add to bashrc
```
export PULSE_SERVER=tcp:$(cat /etc/resolv.conf | grep nameserver | awk '{print $2; exit;}')
```
3. test
```
# test, should output noise on the speakers
pacat < /dev/urandom 
```

### setup wslu and windows browser

* `pacaur -S wslu`: <https://github.com/wslutilities/wslu>
* Add /etc/os-release with <https://github.com/QAston/wslconfig/blob/master/etc/os-release> if not present
* Usage: wslview http://google.com
* set wslview as the default browser:
    * copy .local/share/applications from <https://github.com/QAston/wslconfig/home/.local>
    * run `xdg-settings set default-web-browser wslview.desktop`
    * see <https://wiki.archlinux.org/index.php/Desktop_entries>
* test using `xdg-open http://google.com`

### set up startup script

Make a wsl-init.bat file:
```
wsl -- echo startup done
call start_pulseaudio.bat
start "C:\Program Files\VcXsrv\vcxsrv.exe" multiwin.xlaunch
```

### set up shutdown script

Make a wsl-shutdown.bat file:
```
@rem wsl-shutdown
wsl --shutdown
taskkill  /IM vcxsrv.exe /T /F
taskkill  /IM pulseaudio.exe /T /F
taskkill  /IM "Docker Desktop.exe"  /T /F
```

## setup other linux software

### make a distro id

```
cat <<'EOF' | sudo tee /etc/profile.d/distroid.sh > /dev/null
#!/bin/bash
if [ -n "$WSL_DISTRO_NAME" ]; then
    export DISTRO_ID=ArchContainer
fi
EOF
```

### setup bash

* Follow [bash](../unix/bash.md)

### set up git

* Follow [git](../tools/GIT.md)
* sudo pacman -S kdiff3
* `git config --global diff.tool kdiff3`

### Setup gtk applications
 * `pacman -S lxappearance gnome-themes-standard`
 * Open lxappearance and set theme to dark
    * even if it looks dark by default you should change to white and back, otherwise apps won't recognize it
 * edit.config/gtk-3.0/settings.ini to set gtk-cursor-theme-size=1
 * .bashrc:
    * `export GDK_SCALE="1"`
    * `export GDK_DPI_SCALE=1 # 2 for cygwin xwin`

### Setup kde applications
 * sudo pacman -S qt5ct qt5-xmlpatterns qt5-svg breeze breeze-icons qt5-svg 
 * Set `export QT_QPA_PLATFORMTHEME="qt5ct"` in .bashrc
 * Choose fushion style and breeze icons in qt5ct and apply
 * Breeze style has some widgets broken (scrolling ones)

### setup nix (see tools/nix)

* follow [nix](../tools/nix.md)/setup

### setup vscode

* follow [vscode](../tools/vscode.md)

### setup docker

* follow [docker](../tools/docker.md)/setup wsl2

### set up command line utilities

#### set up ripgrep, fd, jq, sd

```
sudo pacman -S ripgrep fd jq dos2unix zip tldr sd
pacaur -S dust
```

#### set up broot

- follow [broot](../tools/broot.md)

#### set up fzf

- follow [fzf](../tools/fzf.md)/setup_wsl2

#### set up nvim

- follow [neovim](../tools/neovim.md)/setup_wsl2

### set up debugging and tracing

#### set up strace

```
pacman -S strace
# strace <command> to trace syscalls of a command
```

#### set up ltrace

```
pacman -S ltrace
# ltrace <command> to trace .so calls of a command
```

#### set up ftrace
```
pacman -S trace-cmd
# relies on /sys/kernel to be r/w, so needs SYSTEMD_NSPAWN_API_VFS_WRITABLE=1 in env when in nspawn
# trace-cmd list
# trace-cmd start -e <events from list> # kernel wide tracing of events
```

#### set up valgrind

```
sudo pacman -S valgrind
# valgrind <command> runs application and prints memcheck summary
# valgrind --vgdb --vgdb-error 0 <command> - runs command, pauses on first error and allows connecting with dbg to inspect/debug
```

#### set up bpftrace
```
sudo pacman -S bpftrace
# see https://github.com/iovisor/bpftrace for example usageg
```

### set up linux host system and tools which work only in the host system

* follow [linux wsl host](../linux/native_wsl_host.md)

#### set up perf

```
pacman -S perf
# perf stat <command> to trace .so calls of a command
# hardware counters aren't available with wsl2/hyperv enabled :(
```

#### set up rr

```
pacaur -S rr
# rr record <command>
# rr replay <recording> - opens gdb with additonal commands: reverse-stepi, reverse-continue, etc
```

### Setup faster builds - todo

 * ccache: <https://wiki.archlinux.org/index.php/Ccache>
 * Warp: <https://engineering.fb.com/open-source/under-the-hood-warp-a-fast-c-and-c-preprocessor/>
 * Zapcc: <https://github.com/yrnkrn/zapcc>

### Set up custom kernel - todo

 * installing <https://docs.microsoft.com/en-us/windows/wsl/release-notes#build-18945>
 * Building: <https://github.com/kdrag0n/proton_wsl2/blob/v4.19-proton/README.md#installation>

### Set up debugging and tracing software - todo

 * <https://github.com/crash-utility/crash>
 * <https://wiki.archlinux.org/index.php/Benchmarking>

### Other stuff - todo

- https://github.com/4U6U57/wsl-open - for windows
- https://github.com/kitsunyan/xdg-open-server - for native wsl host
- https://github.com/ibraheemdev/modern-unix
- https://direnv.net/
- https://github.com/andreafrancia/trash-cli
- pbcopy/pbpaste
- xdg handlers for file types
* https://github.com/cascadium/wsl-windows-toolbar-launcher
* https://aur.archlinux.org/packages/broot/ https://dystroy.org/broot/ or XTree
* mc ()
    * ctrl-o - switches to shell
    * tab switches left/right
* tmux
* <https://wiki.archlinux.org/index.php/XDG_user_directories#Creating_default_directories>
* <https://aur.archlinux.org/packages/clion/>
* <https://wiki.archlinux.org/index.php/Uniform_look_for_Qt_and_GTK_applications>
* <https://www.freedesktop.org/wiki/Software/xdg-utils/>

## usage

Arch and Artix shouldn't be running at the same time because the init systems change kernel settings differently and the kernel settings are shared

### vhdx 

* wsl vhdx files use a partitionless drive, to create one in hyperv:
    * use new vhdx disk creator, select fixed or dynamic
    * add the vhdx to a vm and initialize it using `sudo mkfs.ext4 /dev/sd<letter>`
* wsl vhdx files have dynamic size by default, capped at 256gb - dynamic type isn't good for linux access
    * converting to fixed will result in 256gb fixed file, that can't be made smaller
    * the only way to make fixed drive smaller is to make a new static drive and then copy contents to the smaller drive using `cp -a` from a vm with access to both drives
        * cd /media/{targetdriveid}
        * sudo cp -a ../{sourcedriveid}/* .
    * make sure to remove the drive from all vms and give everyone read/write access, otherwise you'll get access is denied error on wsl startup
* expanding fixed partitionless drive (theoretical, haven't done this yet):
    * expand the vhdx file in hyperv
    * run [resize2fs](https://man.archlinux.org/man/resize2fs.8.en)

#### mounting additional vhdx files in wsl2

* you can set up a dummy wsl distro to serve as a mountable ext4 drive
* pick a small distro, arch for example
    * if you have arch already, rename the installer executable so the installed distro has a different name
* set up the same users on the dummy distro as in distros you want to use it in
    * `useradd -m -U -G wheel,adm,log,audio,disk,floppy,input,kvm,optical,storage,video dariusza`
* configure /etc/wsl.conf
```
[automount]
enabled=true # Setting to true causes your fixed drives to be mounted undrer /mnt; default=true
crossDistro=true # allow directories, mounts and links created by this distro in /mnt/wsl/* to be visible in other distros;default false
```
* start it up using following script
```
wsl.exe -d DriveArch -u root -e mkdir -p /mnt/wsl/drive && mount --bind /home/dariusza "/mnt/wsl/drive" 
```
* add the above to the wsl bat startup scripts of distros that want this

### wsl2 issues

 * lack of official systemd support - WSL2 uses it’s own init daemon, so systemd commands won’t work. In most cases this can be worked-around by using init.d scripts, for example instead of: `sudo systemctl start ssh` you need to use `/etc/init.d/ssh start`. Some daemons (for example udev) can’t be started this way and there seems to be no workaround for that, but then some of the daemons aren't needed (for example udev)
     * wsl2_artix uses kernel config to change fix this
     * wsl2_arch enables custom systemd support
 * issues releasing memory back to windows - sometimes wsl2 doesn’t free the requested memory properly, there’s a workaround available here ([https://github.com/microsoft/WSL/issues/4166](https://github.com/microsoft/WSL/issues/4166) )
 * there _might_ be issues with opengl rendering or missing opengl features, although I didn’t have any problems so far, in that case the workaround is to use opengl clients built natively on windows
 * cpu performance is slightly worse than native (similar to a vm)
 * the init process can’t be replaced/rebuilt
 * the kernel can be changed/rebuilt as long as it’s compatible with the MS kernel: ([example build](https://github.com/kdrag0n/proton_wsl2/blob/v4.19-proton/README.md#installation)), it’s easy way to swap kernel [using .wslconfig](https://docs.microsoft.com/en-us/windows/wsl/release-notes#build-18945)
    * [example - building with kvm support](https://gist.github.com/offlinehacker/b1d96515f87a47bd0b0bea574eab5583)
 * performance counters aren't working by default, [but can be made to work](https://github.com/microsoft/WSL/issues/4678)
    * [this is how to set the flags in regular hyperv for intel cpus](https://docs.microsoft.com/en-us/windows-server/virtualization/hyper-v/manage/performance-monitoring-hardware)
        * [these options can be edited by intercepting loading of vmcreator](https://gist.github.com/steffengy/62a0b5baa124830a4b0fe4334ccc2606)
    * wsl2 vm is managed using HypervComputeService - HCS API - it's possible perf counters can be enabled from there?
        * [api introduction](https://docs.microsoft.com/en-us/virtualization/community/team-blog/2017/20170127-introducing-the-host-compute-service-hcs)
        * [rust api](https://github.com/rafawo/hcs-rs/blob/c8d70a431eed24ccc53b4119975cc11fc8972d6a/src/schema/virtual_machines/resources/compute/mod.rs)
        * hcsdiag.exe can list the hcs resources
    * what needs to be done for amd performance counters to work? looks like not supported 
* apparently amd uprof [doesn't work when hyperv is enabled](https://community.amd.com/t5/server-gurus-discussions/amd-uprof-not-showing-performance-counters/td-p/438368), this includes in and out of wsl2

### tips

 * making wsl2 coexist with a dns server:  <https://old.reddit.com/r/programming/comments/jidoyn/simple_way_to_docker_on_windows_10_home_with_wsl_2/ga65arg/>
 * you can access windows files from wsl using /mnt/c/ path, this includes starting windows applications from linux command line, this includes running linux executables on windows files
 * you can access linux files from windows using `\\\\wsl$\\<distro-name>\\` path
     * many applications will not like \\\\wsl$\\ path, so as a workaround you can mount the path to a drive (go to \\\\wsl$\\ in explorer, right click on a linux dir and select mount as network drive)
 * [https://github.com/wslutilities/wslu](https://github.com/wslutilities/wslu) - install using sudo apt-get install ubuntu-wsl
 * [https://docs.microsoft.com/en-us/windows/wsl/about](https://docs.microsoft.com/en-us/windows/wsl/about)
 * [https://wiki.ubuntu.com/WSL](https://wiki.ubuntu.com/WSL)
 * Stack Overflow: [https://stackoverflow.com/questions/tagged/wsl](https://stackoverflow.com/questions/tagged/wsl)
 * Ask Ubuntu: [https://askubuntu.com/questions/tagged/wsl](https://askubuntu.com/questions/tagged/wsl)
 * reddit: [https://www.reddit.com/r/bashonubuntuonwindows](https://www.reddit.com/r/bashonubuntuonwindows)
 * [https://www.reddit.com/r/bashonubuntuonwindows/wiki/index](https://www.reddit.com/r/bashonubuntuonwindows/wiki/index)
 * List of programs that work and don't work
     * [https://github.com/ethanhs/WSL-Programs](https://github.com/ethanhs/WSL-Programs)
     * [https://github.com/davatron5000/can-i-subsystem-it](https://github.com/davatron5000/can-i-subsystem-it)
 * Awesome WSL: [https://github.com/sirredbeard/Awesome-WSL](https://github.com/sirredbeard/Awesome-WSL)
 * Tips and guides for new bash users: [https://github.com/abergs/ubuntuonwindows](https://github.com/abergs/ubuntuonwindows)
 * WSLENV for sharing env variables <https://devblogs.microsoft.com/commandline/share-environment-vars-between-wsl-and-windows/> <https://docs.microsoft.com/en-us/windows/wsl/interop#share-environment-variables-between-windows-and-wsl>
 * `/etc/wsl.conf` configuration: <https://github.com/QAston/wslconfig/blob/master/etc/wsl.conf>
    * configures network integration, PATH variable append
    * <https://docs.microsoft.com/en-us/windows/wsl/wsl-config#interop>
* [wsl api in wslapi.dll](https://docs.microsoft.com/en-us/windows/win32/api/wslapi/nf-wslapi-wsllaunch)
* [LxRunOffline](https://github.com/DDoSolitary/LxRunOffline) is a useful tool for manipulating wsl distros, but currently it doesn't work well for wsl2
* [wsldl](https://github.com/yuk7/wsldl) does similar things for wsl2
* [microsoft guide for expanding dynamic vhdx files](https://docs.microsoft.com/en-us/windows/wsl/compare-versions#expanding-the-size-of-your-wsl-2-virtual-hard-disk)
