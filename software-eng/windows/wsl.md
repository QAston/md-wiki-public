WSL 2 is a windows preview feature. It replaces the old WSL system (which was based on linux kernel emulation) with a lightweight VM running a real linux kernel (microsoft has made an open source fork of the linux kernel for this purpose).

### Before you try this out, check out the list of potential issues:
 * lack of systemd support - WSL2 uses it’s own init daemon, so systemd commands won’t work. In most cases this can be worked-around by using init.d scripts, for example instead of: `sudo systemctl start ssh` you need to use `/etc/init.d/ssh start`. Some daemons (for example udev) can’t be started this way and there seems to be no workaround for that
     * if you want coredumps use this: [https://sigquit.wordpress.com/2009/03/13/the-core-pattern/](https://sigquit.wordpress.com/2009/03/13/the-core-pattern/) instead of coredumpctl which is a part of systemd
 * issues releasing memory back to windows - sometimes wsl2 doesn’t free the requested memory properly, there’s a workaround available here ([https://github.com/microsoft/WSL/issues/4166](https://github.com/microsoft/WSL/issues/4166) )
 * there _might_ be issues with opengl rendering or missing opengl features, although I didn’t have any problems so far, in that case the workaround is to use opengl clients built natively on windows
 * WSL2 is a preview feature, which means that some bugs are likely to happen and some software might not support it properly yet
 * cpu performance is slightly worse than native (similar to a vm)
 * the init process can’t be replaced/rebuilt
 * the kernel can be changed/rebuilt as long as it’s compatible with the MS kernel: (example build[https://github.com/kdrag0n/proton\_wsl2/blob/v4.19-proton/README.md#installation](https://github.com/kdrag0n/proton_wsl2/blob/v4.19-proton/README.md#installation)), it’s easy way to swap kernel using .wslconfig: [https://docs.microsoft.com/en-us/windows/wsl/release-notes#build-18945](https://docs.microsoft.com/en-us/windows/wsl/release-notes#build-18945)

### Benefits of using WSL2 over a VM
 * better resource sharing - you don’t need to permamently dedicate half of your pc to a VM
 * better interoperability between host os and linux - you can run windows executables from linux commandline, you can open linux directories with windows explorer, etc.
 * it’s more user friendly - no need to maintain a VM and a separate set of devtools in the vm.

How to
------
1.  Follow the official guide for installing WSL 2: [https://docs.microsoft.com/en-us/windows/wsl/wsl2-install](https://docs.microsoft.com/en-us/windows/wsl/wsl2-install)
    1.  this will require enabling windows insider previews, choose “skip to next windows version” variant if you don’t want to be bothered with updates
2.  Install ubuntu 18.04 as your WSL distribution
3.  Install [https://sourceforge.net/projects/vcxsrv/](https://sourceforge.net/projects/vcxsrv/)
4.  Start the windows X server using XLaunch shortcut on your desktop
    1.  make sure to enable the network access for the application when prompted - it’s used to communicate with wsl
    2.  Select “multiple windows” mode if you want the linux windows to show up on windows bar
    3.  Make sure to change the settings to look like the ones below (disable access control needs to be set and native opengl needs to be **NOT** set - it will actually make the opengl support better for some bizzare reason)
        ![](https://hadean.atlassian.net/wiki/download/attachments/29786128/image-20191007-172448.png?api=v2)
    4.  leave other settings as they are
5.  Configure your linux environment to use the X server (you might want to add this to your bash startup scripts)
    1.  `export __GLX_VENDOR_LIBRARY_NAME`\=mesa
    2.  export LIBGL\_ALWAYS\_INDIRECT=0
    3.  export DISPLAY=$(cat /etc/resolv.conf | grep nameserver | awk '{print $2; exit;}'):0.0
6.  Test X window setup by running
    1.  linux$ sudo apt-get install xterm
    2.  linux$ xterm
    3.  you should see the xterm launch with a gui in windows
7.  Test opengl by running the following:
    1.  linux$ sudo apt-get install mesa-utils
    2.  linux$ glxgears
    3.  if you see the gears window and the gears are turning you have opengl working properly
8.  You should now be able to run aether by following the readme [https://github.com/hadeaninc/aether](https://github.com/hadeaninc/aether) , with minor adjustments:
    1.  make sure you run a branch containing this commit: [https://github.com/hadeaninc/aether/pull/786](https://github.com/hadeaninc/aether/pull/786) (fixes issues with guis under wsl2)
    2.  use `sudo /etc/init.d/ssh start` to start the sshd server
    3.  you might need to manually copy your key to the sshd accepted keys file
    4.  if you need any other daemons, like dbus, they’re started the same way
9.  If you experience memory usage issues, you can try using this workaround: [https://github.com/microsoft/WSL/issues/4166#issuecomment-526725261](https://github.com/microsoft/WSL/issues/4166#issuecomment-526725261)

### Usage tips
 * you can access windows files from wsl using /mnt/c/ path, this includes starting windows applications from linux command line, this includes running linux executables on windows files
 * you can access linux files from windows using \\\\wsl$\\<distro-name>\\ path
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

### Troubleshooting
#### Can’t launch gui applications
 * Make sure you have configured vcxsrv and linux as described above
 * Make sure you have enabled vcxsrv to work in all networks, otherwise linux will not be able to connect to it
     * If GUIs stop working after you’ve started vpn, this is the cause of the problem
     * The correct windows defented configuration looks like the one below (notice All, not public) for VcXsrv!
 * If it still doesn’t work you can try some combination of the following:
     * export DISPLAY=:0
     * set the display number to 0 in vcxsrv settings,
     * you can try a different X server for windows
#### OpenGL applications don’t work, gears don’t turn
 * Make sure you have configured vcxsrv and linux as described above
 * GLFW library has a bug, where glfw\_init() crashes when run in wsl, make sure you use version 3.3 or above where this bug is fixed
 * If it still doesn’t work you can try some combination of the following:
     * Change the vcxsrv to use a single window instead of multi-window mode, opengl support might be better
     * Enable Native OpenGL option in vcxsrv settings, then set LIBGL\_ALWAYS\_INDIRECT=1 env variable in linux
     * unset the `__GLX_VENDOR_LIBRARY_NAME`
     * you can try a different X server for windows
 * Alternatively, if you want to connect to aether, you could build a windows native client instead and connect remotely