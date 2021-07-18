## overview

* [base docs](https://wsldl-pg.github.io/ArchW-docs/How-to-Setup/)

## set up os

1. Download and unpack the zip from <https://github.com/yuk7/ArchWSL>
2. Run the Arch executable to install and unpack
3. Setup users (https://wsldl-pg.github.io/ArchW-docs/How-to-Setup/)
     * Configure root password
     * Configure user: 
     * `useradd -u1000 -m -U -G wheel,adm,log,audio,disk,floppy,input,kvm,optical,storage,video dariusza`
     * `passwd dariusza # set user password`
     * `passwd # set root password`
     * `sudo vim /etc/sudoers # uncomment # %wheel ALL=(ALL) ALL`
     * Change default user in (windows):
     * `Arch.exe config --default-user dariusza`
     * exit root shell
4. Set up pacman
    * set up keys
     * `sudo pacman-key --init`
     * `pacman-key --populate`
    * configure mirrors: <https://wiki.archlinux.org/title/mirrors>
        * uncomment uk mirrors /etc/pacman.d/mirrorlist
        * run `sudo pacman -Syyu`
5. Set up basic packages, follow [neovim](../tools/neovim.md) to configure neovim
```
sudo pacman -S base-devel git wget neovim
```
6. Configure /etc/wsl.conf - copy from wslconfig repo
7. Configure ~/.bash_profile - copy from wslconfig repo


### set up ABS and AUR and other custom packages

- follow [setup ABS and AUR](../linux/arch_custom_packages.md)

### set up windows-linux integration

- follow [wsl](./wsl.md)/setup windows-linux integration

### set up systemd

1. Install https://github.com/qaston/subsystemctl (fixes branch)
    * `./install.sh`
2. Build and install [subsystemctl-bash-wrapper](https://github.com/QAston/wslconfig/tree/master/bin/subsystemctl-redir)
    * run `sudo ./install.sh` from the directory, no more changes are needed
    * to bypass the wrapper run `WSLENV=%WSLENV%:SUBSYSTEMCTL_DISABLE_WRAPPER SUBSYSTEMCTL_DISABLE_WRAPPER=1 wsl.exe`
    * The wrapper will put bash in the systemd context if subsystemctl start has been run, otherwise you run outside
    * todo: the wrapper can also be bypassed running `wsl -d Arch -e <command>` which always invokes /bin/bash to run the command, hard replacing /bin/bash would fix this (need to overwrite bash package to do that and disable updates for it in pacman.conf)
3. Add init to wsl-init.bat script
```
wsl.exe -d Arch -u root -e subsystemctl start && startup done
wt -p Arch -d \\wsl$\Arch\home\dariusza\
```
4. Set up locale
  * Uncomment en_GB.UTF-8 UTF-8 and other needed **[locales](https://wiki.archlinux.org/index.php/Locale)** in `/etc/locale.gen`, and generate them with:
  * `sudo locale-gen`
  * Add `export LANG=en_GB.UTF-8` to `/etc/profile.d/00_settings.sh`
  * Set `/etc/locale.conf LANG=en_GB.UTF-8` 
cat <<'EOF' | sudo tee /etc/locale.conf > /dev/null
LANG=en_GB.UTF-8
EOF
5. Set up journalctl
  * configure smaller log storage
```
sudo mkdir -p /etc/systemd/journald.conf.d/
# sudo tee because sudo doesn't work on > redirection
cat <<'EOF' | sudo tee /etc/systemd/journald.conf.d/00-journal-size.conf > /dev/null
[Journal]
SystemMaxUse=50M
EOF
```
6. Set up kernel params
```
cat <<'EOF' | sudo tee /etc/sysctl.conf > /dev/null
fs.inotify.max_user_watches=524288
EOF
```
7. Remove services which don't work well with wsl
```
sudo mkdir -p /etc/systemd/wsl2-incompatible/user/sockets.target.wants/ /lib/systemd/wsl2-incompatible/system/sysinit.target.wants/ /usr/lib/systemd/wsl2-incompatible/system/
# remove binfmt mounting done by systemd because it doesn't work and triggers 'too many simlinks error
sudo mv /lib/systemd/system/sysinit.target.wants/proc-sys-fs-binfmt_misc.automount /lib/systemd/wsl2-incompatible/system/sysinit.target.wants/
sudo mv /lib/systemd/system/sysinit.target.wants/systemd-binfmt.service /lib/systemd/wsl2-incompatible/system/sysinit.target.wants/
sudo mv /usr/lib/systemd/system/proc-sys-fs-binfmt_misc.automount /usr/lib/systemd/wsl2-incompatible/system/
sudo mv /usr/lib/systemd/system/proc-sys-fs-binfmt_misc.mount /usr/lib/systemd/wsl2-incompatible/system/

#todo: these don't fix anything atm
#sudo mv /etc/systemd/user/sockets.target.wants/dirmngr.socket  /etc/systemd/wsl2-incompatible/user/sockets.target.wants/
#sudo mv /etc/systemd/user/sockets.target.wants/gpg-agent*.socket /etc/systemd/wsl2-incompatible/user/sockets.target.wants/
#sudo mv /lib/systemd/system/sysinit.target.wants/proc-sys-fs-binfmt_misc.mount /lib/systemd/wsl2-incompatible/system/sysinit.target.wants/
#sudo mv /etc/systemd/system/multi-user.target.wants/systemd-resolved.service
#sudo mv /etc/systemd/system/dbus-org.freedesktop.resolve1.service
```
- backout scipts (apply if the scripts turn out to be useful)
```
sudo mv /etc/systemd/wsl2-incompatible/user/sockets.target.wants/dirmngr.socket   /etc/systemd/user/sockets.target.wants/
sudo mv /etc/systemd/wsl2-incompatible/user/sockets.target.wants/gpg-agent*.socket /etc/systemd/user/sockets.target.wants/
sudo mv /lib/systemd/wsl2-incompatible/system/sysinit.target.wants/systemd-binfmt.service /lib/systemd/system/sysinit.target.wants/
sudo mv  /lib/systemd/wsl2-incompatible/system/sysinit.target.wants/proc-sys-fs-binfmt_misc.automount /lib/systemd/system/sysinit.target.wants/
sudo mv /usr/lib/systemd/wsl2-incompatible/system/proc-sys-fs-binfmt_misc.automount  /usr/lib/systemd/system/
sudo mv /usr/lib/systemd/wsl2-incompatible/system/proc-sys-fs-binfmt_misc.mount  /usr/lib/systemd/system/
```


### set up other linux software

- follow [wsl](./wsl.md)/setup other linux software
