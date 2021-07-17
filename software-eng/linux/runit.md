## Runit and other config for replacing systemd

### Docs/Overview

 * <https://kchard.github.io/runit-quickstart/>
 * <https://wiki.artixlinux.org/Main/Runit>
 * Runit manages services in `/etc/runit/runsvdir`
 * Runit manages system startup/shutdown 
     * currently unused in WSL2 because wsl's init does most these things already and because we're not running the `runit-init` command on startup
    * `/etc/runit/1` - first startup stript, script need to finish
        * runs `/etc/rc/sysinit/*`
        * not run because wsl2's init does these already (afaik)
        * the only useful script seems to be: /etc/rc/sysinit/85-dmesg which just stores output of dmesg to dmesg.log at startup
    * `/etc/runit/2` - script started after 1, continues to run until shutdown - runs `/etc/runit/runsvd services` and `/etc/local.d/*`
        * perhaps /etc/runit/2 is the script to run on startup?
    * `/etc/runit/3` - shutdown scripts
 * documentation is spread around the manual pages for commands
    * <http://smarden.org/runit/runsv.8.html>
        * commands to manipulate services
        * service/run script defines the service
        * service/log directory contains a logging service defintion, if any log handling is needed
            * if no log directory is defined, stdout and stderr of running services will be shown in output of runsvdir
            * the defined /log service will receive stdout by default, merge stderr into stdout (`exec 2>&1` in run for example) to make sure it gets everything
    * <http://smarden.org/runit/svlogd.8.html>
        * logger writer/filter/rotator implementation designed to work with runit
    * <http://smarden.org/socklog2/configuration.html>
        * runit-service that can poll logs from various sources and forwards them to svlogd using runit's ./log mechanism
        * socklog-unix - /dev/log
        * socklog-klog - /proc/kmsg (dmsg)
    * <http://smarden.org/runit/runsvdir.8.html>
        * `log` argument to runsvdir will show in `ps aux` argument line for runsvdir and will be replaced by errors if any
 * `logger` executable is a part of [util-linux](https://en.wikipedia.org/wiki/Util-linux), it adds logs to system logger if set up (/var/log/syslog? dev/log?) <https://man.archlinux.org/man/logger.1.en>
 * syslog-ng is a logging daemon that reads from /dev/log (which it sets up), dmsg, sockets, and writes to configured locations
 * use kernel dump config instead of coredumpctl: <http://man7.org/linux/man-pages/man5/core.5.html> and <https://sigquit.wordpress.com/2009/03/13/the-core-pattern/>
    * kernel config change from a container will affect the entire machine

### Usage

  * ls /etc/runit/sv/ # see possible services
  * ln -s /etc/runit/sv/service_name /etc/runit/runsvdir/startwsl # add to default runlevel
  * sudo runsvdir /etc/runit/runsvdir/startwsl & # run services linked to the directory
  * sudo sv [service dir] # run single service
  * `sudo sv status /etc/runit/runsvdir/startwsl/sshd` - show service status, must be in /run/runit/service to work without full path
  * ls /etc/rc/sysinit/ # see startup scripts that aren't executed in wsl because wsl has it's own init system
  * dmesg - prints kernel messages from kernel's log ring buffer


### Setup

1. Setup runit
* create a dir for wsl runners
```
# create a directory for wsl specific services (so that default services don't get restored on update)
sudo mkdir /etc/runit/runsvdir/startwsl
```
* add init code from /etc/runit/2 to ~/bin/init
```
runsvchdir "startwsl"

rm -rf /run/runit
mkdir -p /run/runit
ln -s /etc/runit/runsvdir/current /run/runit/service

exec env - PATH=$PATH \
runsvdir -P /run/runit/service
```
2. Install services
  * `pacman -S dbus dbus-runit openssh openssh-runit`
  * For dbus run: `dbus-uuidgen > /etc/machine-id` # post install
  * `sudo ln -s  /etc/runit/sv/sshd /etc/runit/runsvdir/startwsl/`
3. Set up a systemd coredumpctl replacement
  * Configure according to 
  * `mkdir -p /tmp/cores`
  * `chmod a+rwx /tmp/cores`
  * `sysctl -w kernel.core_pattern=/tmp/cores/%E!!!.%P.%t.sig%s.thrd%I.%h` - execute on startup
  * Add to bashrc: `ulimit -c unlimited`
  * See <https://github.com/QAston/wslconfig/blob/master/home/bin/gdbdump> for running dumps
  * bt full in gdb prints stack frames content
  * core command can be run to get a dump of a running process
4. Add init to the wsl-init.bat script
```
wt -p Artix -d \\wsl$\Artix\home\dariusza wsl.exe -u root -e ./bin/init; -d \\wsl$\Artix\home\dariusza\ -p Artix
```
