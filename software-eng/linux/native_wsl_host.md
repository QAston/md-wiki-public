## native wsl host

- makes wsl into an environment which can be used both in and out of hyperv
- using outside of hyperv enables tools which don't work in hyperv like rr and amd hardware perf counters
- because the wsl2 devenv is accessible from both windows and linux, the environment is much less likely to get bitrotten
- for this to work, wsl2 must not have hard dependencies on the windows or linux host, or it must be able to depend on either of these
- things that are host-specific need to be guarded by WSL_DISTRO_NAME env variable existence check
- if anything needs sharing between the system put it in wsl

### setup on windows

#### change windows to use utc bios time

#### convert the image to a fixed vhdx

- open the file in hyperv manager  (right click on a vm -> edit image)
- convert the dynamic image to fixed one
- once converted, qemu-nbd will stop marking the image as sparse!

#### disable fast boot and hibernation

- power and sleep settings -> advanced settings -> enable unavailable options -> deselect turn on fast statup -> save changes
- in admin cmd:
```
powercfg.exe /hibernate off
```
- turning off hibernation should disable hybrid sleep, but you can check for it just in case in power plan advanced options

#### create a partition for manjaro (native host system)

- the partition editor is a bit dumb, create it in partition settings instead, doesn't need to be big as the host system doesn't need much space

### setup wsl2 vhdx contents

- set up env variables
```
if [ -n "$WSL_DISTRO_NAME" ]; then
# old vars here
else
export DISPLAY=:0
export PULSE_SERVER=tcp:127.0.0.1
unset LIBGL_ALWAYS_INDIRECT
#export __NV_PRIME_RENDER_OFFLOAD=0
#export __GL_SYNC_TO_VBLANK=1
#export __GLX_VENDOR_LIBRARY_NAME=nvidia
#export LIBGL_DEBUG=verbose
fi
```
- add an etc/resolv.conf service (Arch only)
```
cat << 'EOF' | sudo tee /root/wslinit > /dev/null
#!/bin/bash
if [ -n "$WSL_DISTRO_NAME" ]; then
        ln -sf /run/resolvconf/resolv.conf /etc/resolv.conf
else
        ln -sf /mnt/host/etc/resolv.conf /etc/resolv.conf
        # link to linux graphics drivers on host machine
        ln -sf /mnt/host/dev/dri/ /dev/dri
fi
EOF
sudo chmod a+x /root/wslinit

cat << 'EOF' | sudo tee /etc/systemd/system/wslinit.service > /dev/null
[Unit]
Description=Wsl2 init service

[Service]
Type=oneshot
ExecStart=/root/wslinit
Restart=no

[Install]
WantedBy=local-fs.target
EOF

# enable autostart
sudo systemctl enable wslinit.service
```

### make a distro id for display on the host

```
cat <<'EOF' | sudo tee /etc/profile.d/distroid.sh > /dev/null
#!/bin/bash
if ! [ -n "$WSL_DISTRO_NAME" ]; then
    export DISTRO_ID=ArchContainer
fi
EOF
```

### setup manjaro (native host system)

- install manjaro from a usb drive
    - gb keyboard settings
    - gb language and timezone
    - propertiary drivers
- update packages
- configure display settings for 2 monitors
    - search for display settings/manager
- install packages
```
sudo pacman -S neovim xclip base-devel noto-fonts-emoji
pacaur -S noto-color-emoji-fontconfig
```
- set up mounting tools for vhd
```
sudo pacman -S qemu-headless nbd
```
- load nbd module on startup:
```
echo "nbd" | sudo tee /etc/modules-load.d/nbd.conf > /dev/null
echo "options nbd max_part=16" | sudo tee /etc/modprobe.d/nbd.conf > /dev/null
```
- load br_netfilter on startup
```
echo "br_netfilter" | sudo tee /etc/modules-load.d/br_netfilter.conf > /dev/null
```
- configure nbd to not use sparse files
```
# sudo nvim /etc/nbd-server/config
# add in [export] sections
sparse_cow = false
```
- make mount directories for vhdx images
```
mkdir ~/wsl2-vhd/
mkdir ~/wsl2-docker-vhd/
```
- restart to load the kernel modules

#### mount the ntfs drive

- create mount points
```
mkdir ~/wsl2-ntfs
```
- find partition uuid of /dev/sdc2 ("Linux" label)
```
lsblk -f
```
- add to fstab
```
UUID=<uuid from lsblk> /home/dariusza/wsl2-ntfs ntfs-3g auto,rw,defaults,noatime,uid=1000,gid=1000,umask=022,utf8 0 0
```

#### set up audio/video support

- install drivers (nvidia machines)
```
sudo pacman -S nvidia-libgl
```
- configure x server - in `sudo nano /etc/sddm.conf`
```
# add +iglx option
ServerArguments=-nolisten tcp +iglx
```
- allow connecting to the x server from the container
```
echo "xhost +local:" | sudo tee /etc/profile.d/03_x11wsl.sh > /dev/null
```
- configure pulseaudio - in `sudo nvim /etc/pulse/default.pa`
```
# add ip auth for the host
load-module module-native-protocol-tcp auth-ip-acl=127.0.0.1
```

#### setup a systemd service to mount/unmount and start main container

- create a mount script
```
cat <<'EOF' > /home/dariusza/mount-wsl2.sh
#!/bin/bash

set -e

VHDX_IMG="$1"
MOUNT_POINT="$2"
NBD_DEVICE="$3"
MACHINECTL_SERVICE="$4"

shift 4

function unmount_device {
  qemu-nbd -d $NBD_DEVICE
  echo "cleanup done"
}

function unmount_partition {
  umount "$MOUNT_POINT"
}

function unmount_all {
  echo "unmount_all"
  unmount_partition
  unmount_device
}

function shutdown_all {
  echo "shutdown_all"
  machinectl stop $MACHINECTL_SERVICE || true
  unmount_all
}

# mount block device
qemu-nbd --detect-zeroes=off -c $NBD_DEVICE --image-opts driver=vhdx,file.filename=$VHDX_IMG

trap unmount_device EXIT

# reload partition table
partprobe $NBD_DEVICE

# mount partition
mount -o rw,nouser $NBD_DEVICE "$MOUNT_POINT"
trap unmount_all EXIT

mkdir -p /mnt/wsl
chmod 777 /mnt/wsl

systemd-notify --ready

trap shutdown_all EXIT

# SYSTEMD_NSPAWN_API_VFS_WRITABLE=1 - make kernel filesystems writeable - needed for docker in a container
# SYSTEMD_SECCOMP=0 - disable capability checks
# SYSTEMD_NSPAWN_USE_CGNS=0 - https://wiki.archlinux.org/title/systemd-nspawn#Run_docker_in_systemd-nspawn
SYSTEMD_SECCOMP=0 systemd-nspawn -M "$MACHINECTL_SERVICE" -D "$MOUNT_POINT" --bind=/mnt/wsl:/mnt/wsl --bind=/:/mnt/host --bind-ro=/tmp/.X11-unix --capability=all "$@"

# --resolv-conf=bind-host - binds too early before the file is populated by network manager
EOF
chmod a+x /home/dariusza/mount-wsl2.sh
```
- add wsl container mount service
```
WSL_INIT_SCRIPT=--boot
DOCKER_FS_PATH=/ # Native host
WSL_DISTRO_PATH=/home/dariusza/wsl2-ntfs/Arch/ext4.vhdx

cat << EOF | sudo tee /etc/systemd/system/mount-wsl2.service > /dev/null
[Unit]
Description=Wsl2 mount service
RequiresMountsFor=/home/dariusza/wsl2-ntfs /tmp /dev/dri
Requires=systemd-modules-load.service
After=systemd-modules-load.service

[Service]
Type=notify
ExecStart=/home/dariusza/mount-wsl2.sh "${WSL_DISTRO_PATH}" "/home/dariusza/wsl2-vhd/" "/dev/nbd1" wsl2-vhd  --bind=/mnt/wsl/arch/root:/mnt/wsl/arch/root --bind=${DOCKER_FS_PATH}home/dariusza/docker-bin:/mnt/wsl/host-arch/bin --bind=${DOCKER_FS_PATH}:/mnt/wsl/host-arch/root ${WSL_INIT_SCRIPT}
Restart=no

[Install]
WantedBy=local-fs.target
EOF

# enable autostart
sudo systemctl enable mount-wsl2.service
```
- set up a login shell script (Arch)
```
DOCKER_SUFFIX="" # DOCKER_SUFFIX="-docker" for the docker container
cat << EOF | gcc -o /home/dariusza/bash-wsl2${DOCKER_SUFFIX}.sh -xc -
#include <unistd.h>
int main(int argc, char** argv) {
  return execlp("machinectl", "machinectl", "shell", "--quiet", "--uid", "1000", "wsl2-vhd$DOCKER_SUFFIX", (char *) NULL);
}
EOF
sudo chown root /home/dariusza/bash-wsl2${DOCKER_SUFFIX}.sh
sudo chmod u+sw,a+rx /home/dariusza/bash-wsl2${DOCKER_SUFFIX}.sh
```
- start the created services (or reboot)
```
sudo mount ~/wsl2-ntfs
sudo subsystemctl start mount-wsl2.service
```
- [configure the host to mimic the wsl2 container host](../windows/wsl2_host_container.md)

#### configure the host system

- make a distro id
```
cat <<'EOF' | sudo tee /etc/profile.d/distroid.sh > /dev/null
#!/bin/bash
if ! [ -n "$WSL_DISTRO_NAME" ]; then
    export DISTRO_ID=ManjaroHost
fi
EOF
```
- copy configuration from wsl
```
cp -ar ~/wsl2-vhd/home/dariusza/.ssh/ .
cp -r ~/wsl2-vhd/home/dariusza/.git* ~
git clone git@github.com:QAston/wslconfig.git
ln -s /home/dariusza/wslconfig/home/bin/ bin
cd ~/wslconfig
./native-host-install.bash
sudo pacman -S broot fzf bash-completion
```
- set up host sysctl config from host.conf
```
# arch-only:
sudo cp /home/dariusza/wsl2-vhd/etc/sysctl.conf /etc/sysctl.d/60-wslhost.conf
sudo sysctl -p /etc/sysctl.d/60-wslhost.conf
```
- configure konsole and yakuake
  - change appearance to use breeze-silver scheme
  - add a profile that runs ~/bash-wsl2.sh as the shell and make it the default

### todos

- use nspawn and mount config files instead of a script service?
  - could use blockdev@.target for /dev/dri? and maybe ndb?
  - ndb client should be configured using ndbtab and connect to qemu-nbd using a unix socket
    - this would probably refresh things on sleep?
- faster io on linux by moving the image out of vhdx into a real ext4 partition which can be simply mounted:
    - https://docs.microsoft.com/en-us/windows/wsl/wsl2-mount-disk (currently in insider only)
      - would need to make a "host" distro that chroots into the mounted drive?
        - a dummy distro with just a fake /bin/bash would work
          - ideally would only depend on the kernel
          - i'd need to invoke chroot to set the paths properly
            - could use the executable on the target distro's drive
        - make docker distro a "host distro"?
          - probably bad, would have to always start docker's bash to get to wsl itself
    - would have to make a usb stick with ext4 to move files between machines and make backups
    - alternatively try to make ext4 partition be visible as a special vhdx file? by somehow implementing [device files](https://en.wikipedia.org/wiki/Device_file) on windows? possibly by implementing it in linux and loading wsl partition from wsl share?
    - linux kernel should get the new ntfs driver realtively soon, hopefully that'd make things run faster?
- todo; qemu-nbd shuts down on sleep, this doesn't work well with laptops, so need to unpack the image instead
  - set up nbd server to prevent this?
    - qemu-nbd works both as client and server in the above setup
    - maybe running it as a server and running separate nbd-client process would help?
    - sleep is done through systemctl suspend
      - systemctl-suspend.service
- cleanup links/mounts between host and the container
  - replace some symlinks with mount --bind (mounts parent directory handling is better)
    - can also configure sharing using --make-shared and --make-rshared, see <https://linux.die.net/man/8/mount>
  - rename chorusone dir to host-workspace
   
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
