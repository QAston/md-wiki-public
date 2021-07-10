## overview

* [base docs](https://wsldl-pg.github.io/ArchW-docs/How-to-Setup/)

## set up os

1. Download and unpack the zip from <https://github.com/yuk7/ArchWSL>
2. Run the Arch executable to install and unpack
3. Setup users (https://wsldl-pg.github.io/ArchW-docs/How-to-Setup/)
     * Configure root password
     * Configure user: 
     * `useradd -m -U -G wheel,adm,log,audio,disk,floppy,input,kvm,optical,storage,video dariusza`
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
sudo pacman -S base-devel git subversion openssh wget neovim
```
6. Configure /etc/wsl.conf:
```
[interop]
appendWindowsPath=false # disable windows path in wsl
```
7. Configure ~/.bash_profile
```
# add windows path setup
if ! [ -z "$PATH_APPEND_BASH_COMMON" ]; then
PATH="$PATH:$PATH_APPEND_BASH_COMMON"
fi

if ! [ -z "$PATH_VAR_ALL" ]; then
PATH="$PATH:$PATH_VAR_ALL"
fi
```

### set up ABS and AUR and other custom packages

- follow [setup ABS and AUR](../linux/arch_custom_packages.md)

### set up windows-linux integration

- follow [wsl](./wsl.md)/setup windows-linux integration

### set up systemd

1. Install https://github.com/sorah/subsystemctl
    * makepkg -s 
2. Build and install [subsystemctl-bash-wrapper](https://github.com/QAston/wslconfig/tree/master/bin/subsystemctl-redir)
    * add bash to ignored pacman packages in /etc/pacman.conf
    * The wrapper will put bash in the systemd context if subsystemctl start has been run, otherwise you run outside
3. Add init to wsl-init.bat script
```
wsl.exe -d Arch -u root -e subsystemctl start && startup done
wt -p Arch -d \\wsl$\Arch\home\dariusza\
```
4. Set up locale
  * Uncomment en_GB.UTF-8 UTF-8 and other needed **[locales](https://wiki.archlinux.org/index.php/Locale)** in `/etc/locale.gen`, and generate them with:
  * `sudo locale-gen`
  * Add `export LANG=en_GB.UTF-8` to `.bashrc`
  * Set `/etc/locale.conf LANG=<em>en_GB.UTF-8`
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

### set up other linux software

- follow [wsl](./wsl.md)/setup other linux software
