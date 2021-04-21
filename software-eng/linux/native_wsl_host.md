## native wsl host

- makes wsl into an environment which can be used both in and out of hyperv
- using outside of hyperv enables tools which don't work in hyperv like rr and amd hardware perf counters
- because the wsl2 devenv is accessible from both windows and linux, the environment is much less likely to get bitrotten
- for this to work, wsl2 must not have hard dependencies on the windows or linux host, or it must be able to depend on either of these
- things that are host-specific need to be guarded by WSL_DISTRO_NAME env variable existence check
- if anything needs sharing between the system put it in wsl

### usage

connect to wsl:
```
ssh localhost 
```

#### manually desparse a file

- qemu-nbd can make the drive file sparse
- run `fsutil sparse getflag "F:\Artix\ext4.vhdx"` to make check if file is made sparse
  - if it is, make a copy of the file
  - then run `fsutil sparse setflag "F:\Artix\ext4.vhdx" 0`

### setup on windows

#### disable fast boot and hibernation

- power and sleep settings -> advanced settings -> enable unavailable options -> deselect turn on fast statup -> save changes
- in admin cmd:
```
powercfg.exe /hibernate off
```
- turning off hibernation should disable hybrid sleep, but you can check for it just in case in power plan advanced options

#### create a partition for manjaro

- the partition editor is a bit dumb, create it in partition settings instead, doesn't need to be big as the host system doesn't need much space

### setup wsl2 vhdx contents

- set up drivers
```
pacman -S nvidia-libgl
```
- set up sshd
```
cat ~/.ssh/id_rsa.pub > ~/.ssh/authorized_keys
```
- set up env variables
```
if [ -n "$WSL_DISTRO_NAME" ]; then
# old vars here
else
export DISPLAY=:0
export PULSE_SERVER=tcp:127.0.0.1
unset LIBGL_ALWAYS_INDIRECT
export __NV_PRIME_RENDER_OFFLOAD=0
export __GL_SYNC_TO_VBLANK=1
export __GLX_VENDOR_LIBRARY_NAME=nvidia
export LIBGL_DEBUG=verbose
fi
```

### setup manjaro

- install manjaro from a usb drive
    - gb keyboard settings
    - gb language and timezone
    - propertiary drivers
- update packages
- configure display settings for 2 monitors
    - search for display settings/manager
- configure konsole and yakuake to use breeze-silver scheme
- copy ssh keys
```
mkdir ~/.ssh
paste keys from backup
chmod 700 ~/.ssh
chmod 0600 /home/dariusza/.ssh/id_rsa
```
- set up convenience links
```
ln -s ~/wsl2-vhd/home/dariusza ~/home-wsl
```
- set up clock sync with windows
```
sudo timedatectl set-local-rtc true # set windows clock
```
- set up mounting tools for vhd
```
sudo pacman -S qemu-headless nbd
```
- load nbd module on startup - run in su session:
```
echo "nbd" > /etc/modules-load.d/nbd.conf
echo "options nbd max_part=16" > /etc/modprobe.d/nbd.conf
```
- configure nbd to not use sparse files
```
# sudo nvim /etc/nbd-server/config
# add in [export] sections
sparse_cow = false
```
- set up basic utilities by copying the wsl config
```
cp wsl2-vhd/home/dariusza/.inputrc ~/.inputrc
cp wsl2-vhd/home/dariusza/.bashrc ~/.bashrc
nano ~/.bashrc # remove settings which depend on utilities
sudo pacman -S neovim
```
- set up host sysctl config from host.conf
```
sudo cp wsl2-vhd/home/dariusza/bin/host.conf /etc/sysctl.d/60-wslhost.conf
sudo sysctl -p /etc/sysctl.d/60-wslhost.conf
mkdir /tmp/cores
```

#### mount the ntfs drive

- create mount points
```
sudo mkdir wsl2-ntfs
```
- find partition uuid of /dev/sdc2 ("Linux" label)
```
lsblk -f
```
- add to fstab
```
UID=AC481B43481B0BA8 /home/dariusza/wsl2-ntfs ntfs-3g auto,rw,defaults,uid=1000,gid=1000,umask=022,utf8 0 0
```

#### set up audio/video support

- install drivers
```
sudo pacman nvidia-libgl
```
- configure x server - in `sudo nano /etc/sddm.conf`
```
# add +iglx option
ServerArguments=-nolisten tcp +iglx
```
- allow connecting to the x server from the container
```
echo "xhost +local:" >> ~/.bash_profile
```
- configure pulseaudio - in `sudo nano /etc/pulse/default.pa`
```
# add ip auth for the host
load-module module-native-protocol-tcp auth-ip-acl=127.0.0.1
```

#### setup a systemd service to mount/unmount and start container

- nano ~/mount-wsl2.sh to start wsl daemon
```
#!/bin/bash
set -e

function unmount_device {
  qemu-nbd -d /dev/nbd0
  echo "cleanup done"
}

function unmount_partition {
  umount "$MOUNT_POINT"
}

function unmount_all {
  unmount_partition
  unmount_device
}

VHDX_IMG="/home/dariusza/wsl2-ntfs/Artix/ext4.vhdx"
MOUNT_POINT="/home/dariusza/wsl2-vhd/"

# mount block device
qemu-nbd --detect-zeroes=off -c /dev/nbd0 "$VHDX_IMG"
trap unmount_device EXIT

# reload partition table
partprobe /dev/nbd0

# mount partition
mount -o rw,nouser /dev/nbd0 "$MOUNT_POINT"
trap unmount_all EXIT

#SYSTEMD_NSPAWN_API_VFS_WRITABLE=1 - make kernel filesystems writeable
SYSTEMD_SECCOMP=0 systemd-nspawn -D "$MOUNT_POINT" --bind=/:/mnt/host --bind-ro=/tmp/.X11-unix --capability=all /home/dariusza/bin/init
```
- sudo nvim /etc/systemd/system/mount-wsl2.service
```
[Unit]
Description=Wsl2 mount service
RequiresMountsFor=/home/dariusza/wsl2-ntfs /tmp
Requires=systemd-modules-load.service
After=systemd-modules-load.service

[Service]
ExecStart=/home/dariusza/mount-wsl2.sh
Restart=no

[Install]
WantedBy=local-fs.target
```
- enable service autostart
```
sudo systemctl enable mount-wsl2.service
```

### todos

- docker? run on the host instead?
- faster io on linux by moving the image out of vhdx into a real ext4 partition which can be simply mounted:
    - https://docs.microsoft.com/en-us/windows/wsl/wsl2-mount-disk
    - alternatively try to make ext4 partition be visible as a special vhdx file? by somehow implementing [device files](https://en.wikipedia.org/wiki/Device_file) on windows? possibly by implementing it in linux and loading wsl partition from wsl share?
- solve the sparse image problem by not using wsl's vhd as a linux entry point?
  - need a small distro to bootstrap things and start a qemu image
    - needs to forward kernel filesystems and do other stuff
    - would be neat if nspawn could be used but can't
- solve the sparse image problem by making qemu not write sparse files
  - `--image-opts driver=vhdx,file.filename=ext4.vhdx,file.driver=vhdx`
  - <https://qemu-project.gitlab.io/qemu/system/images.html#disk-image-file-formats>
  - <https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=934873>
- solve the sparse problem by not allowing writes from linux
  - qemu-nbd --snapshot flag
- solve the sparse problem by filling in the sparse gaps
  - takes 20 minutes :(
- solve the sparse problem by making a fork that doesn't use sparse filesystem calls
  - https://github.com/qemu/qemu/blob/266469947161aa10b1d36843580d369d5aa38589/block/vhdx-log.c

#### building a fork of qemu-nbd
pacaur -S asp

mkdir ~/abs
cd ~/abs
asp checkout qemu-headless

cd qemu
cd trunk
todo: add a path file

```
# add patch to PKGBUILD
nano PKGBUILD
```
makepkg -s --skipchecksums --skipinteg --skippgpcheck

```
# disable pgp check in /etc/pacman.conf
SigLevel    = Never
#Required DatabaseOptional
LocalFileSigLevel = Never
#Optional
```

sudo pacman -U qemu-headless-5.2.0-5-x86_64.pkg.tar.zst

```
# reenable pgp check in pacman.conf, add qemu-headless to ignored packages
IgnorePkg   = qemu-headless
SigLevel    = Required DatabaseOptional
LocalFileSigLevel = Optional
```
        

### references

- <https://wiki.archlinux.org/index.php/chroot>
- <https://wiki.archlinux.org/index.php/Systemd-nspawn>
    - don't use machinectl, it doesn't work with children which don't have systemd
    - by default nspawn shares the network of the host (which is perfect), unless any network is specified
    - -M specifies the name of the machine, which can be used in other systemctl commands and as a hostname of the container
        - by default the machine name is the name of the directory in which root is placed
    - -u specifies the starting user, -E env variables, --chdir workdir
    - any spare arguments are the starting command
    - systemd-run -M <command> can be used to execute commands in a running container

- <http://manpages.ubuntu.com/manpages/bionic/man1/schroot.1.html>
