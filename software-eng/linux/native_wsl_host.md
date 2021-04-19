## native wsl host

- makes wsl into an environment which can be used both in and out of hyperv
- using outside of hyperv enables tools which don't work in hyperv like rr and amd hardware perf counters
- because the wsl2 devenv is accessible from both windows and linux, the environment is much less likely to get bitrotten
- for this to work, wsl2 must not have hard dependencies on the windows or linux host, or it must be able to depend on either of these
- things that are host-specific need to be guarded by WSL_DISTRO_NAME env variable existence check
- if anything needs sharing between the system put it in wsl

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
export DISPLAY=:0
export PULSE_SERVER=tcp:127.0.0.1
unset LIBGL_ALWAYS_INDIRECT
export __NV_PRIME_RENDER_OFFLOAD=0
export __GL_SYNC_TO_VBLANK=1
export __GLX_VENDOR_LIBRARY_NAME=nvidia
export LIBGL_DEBUG=verbose
```
- set up drivers

### setup manjaro

- install manjaro from a usb drive
    - gb keyboard settings
    - gb language and timezone
    - propertiary drivers
- update packages
- configure display settings for 2 monitors
    - search for display settings/manager
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
- configure pulseaudio - in `sudo nano /etc/pulse/default.pa`
```
# add ip auth for the host
load-module module-native-protocol-tcp auth-ip-acl=127.0.0.1
```

#### mount/unmount and start container

### mount the vhd drive

- mount script - requires sudo
```
!/bin/bash

VHDX_IMG="wsl2-ntfs/Artix/ext4.vhdx"
MOUNT_POINT="wsl2-vhd/"

# Load the nbd kernel module.
rmmod nbd;modprobe nbd max_part=16

# mount block device
qemu-nbd -c /dev/nbd0 "$VHDX_IMG"

# reload partition table
partprobe /dev/nbd0

# mount partition
mount -o rw,nouser /dev/nbd0 "$MOUNT_POINT"
```
- unmount
```
MOUNT_POINT="wsl2-vhd/"
umount "$MOUNT_POINT" && qemu-nbd -d /dev/nbd0 && rmmod nbd
```
- start
```
cd ~/wsl2-vhd
sudo systemd-nspawn --bind-ro=$HOME/.Xauthority:/root/.Xauthority  --bind-ro=/tmp/.X11-unix
```
- login into shell
```
ssh localhost
```

### todos

- automate startup/shutdown
- docker? run on the host instead?
- faster io on linux by moving the image out of vhdx into a real ext4 partition which can be simply mounted:
    - https://docs.microsoft.com/en-us/windows/wsl/wsl2-mount-disk

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