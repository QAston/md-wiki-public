WSL 2 is a windows preview feature. It replaces the old WSL system (which was based on linux kernel emulation) with a lightweight VM running a real linux kernel (microsoft has made an open source fork of the linux kernel for this purpose).

### wsl2 issues
 * lack of systemd support - WSL2 uses it’s own init daemon, so systemd commands won’t work. In most cases this can be worked-around by using init.d scripts, for example instead of: `sudo systemctl start ssh` you need to use `/etc/init.d/ssh start`. Some daemons (for example udev) can’t be started this way and there seems to be no workaround for that
     * if you want coredumps use this: [https://sigquit.wordpress.com/2009/03/13/the-core-pattern/](https://sigquit.wordpress.com/2009/03/13/the-core-pattern/) instead of coredumpctl which is a part of systemd
 * issues releasing memory back to windows - sometimes wsl2 doesn’t free the requested memory properly, there’s a workaround available here ([https://github.com/microsoft/WSL/issues/4166](https://github.com/microsoft/WSL/issues/4166) )
 * there _might_ be issues with opengl rendering or missing opengl features, although I didn’t have any problems so far, in that case the workaround is to use opengl clients built natively on windows
 * cpu performance is slightly worse than native (similar to a vm)
 * the init process can’t be replaced/rebuilt
 * the kernel can be changed/rebuilt as long as it’s compatible with the MS kernel: ([example build](https://github.com/kdrag0n/proton_wsl2/blob/v4.19-proton/README.md#installation)), it’s easy way to swap kernel using .wslconfig: [https://docs.microsoft.com/en-us/windows/wsl/release-notes#build-18945](https://docs.microsoft.com/en-us/windows/wsl/release-notes#build-18945)

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
 * WSLENV for sharing env variables <https://devblogs.microsoft.com/commandline/share-environment-vars-between-wsl-and-windows/> <https://docs.microsoft.com/en-us/windows/wsl/interop#share-environment-variables-between-windows-and-wsl>
 * `/etc/wsl.conf` configuration: <https://github.com/QAston/wslconfig/blob/master/etc/wsl.conf>
    * configures network integration, PATH variable append
    * <https://docs.microsoft.com/en-us/windows/wsl/wsl-config#interop>

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
 #### wsl2 conflicting with dns server
 * <https://old.reddit.com/r/programming/comments/jidoyn/simple_way_to_docker_on_windows_10_home_with_wsl_2/ga65arg/>