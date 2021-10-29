## native wsl host

- makes wsl into an environment which can be used both in and out of hyperv
- using outside of hyperv enables tools which don't work in hyperv like rr and amd hardware perf counters
- because the wsl2 devenv is accessible from both windows and linux, the environment is much less likely to get bitrotten
- for this to work, wsl2 must not have hard dependencies on the windows or linux host, or it must be able to depend on either of these
- things that are host-specific need to be guarded by WSL_DISTRO_NAME env variable existence check
- if anything needs sharing between the system put it in wsl

### setup on windows

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

- set up drivers
```
pacman -S nvidia-libgl
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
- add an etc/resolv.conf service (Arch only)
```
cat << 'EOF' | sudo tee /root/wslinit > /dev/null
#!/bin/bash
if [ -n "$WSL_DISTRO_NAME" ]; then
        ln -sf /run/resolvconf/resolv.conf /etc/resolv.conf
else
        ln -sf /mnt/host/etc/resolv.conf /etc/resolv.conf
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
- set up clock sync with windows
```
sudo timedatectl set-local-rtc 1 # set windows clock, cat /etc/adjtime to verify
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
- load br_netfilter on startup
```
echo "br_netfilter" > /etc/modules-load.d/br_netfilter.conf
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
UID=AC481B43481B0BA8 /home/dariusza/wsl2-ntfs ntfs-3g auto,rw,defaults,uid=1000,gid=1000,umask=022,utf8 0 0
```

#### set up audio/video support

- install drivers
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
echo "xhost +local:" > /etc/profile.d/03_x11wsl.sh
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

shift 3

function unmount_device {
  qemu-nbd -d $NBD_DEVICE
  echo "cleanup done"
}

function unmount_partition {
  umount "$MOUNT_POINT"
}

function unmount_all {
  unmount_partition
  unmount_device
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

# SYSTEMD_NSPAWN_API_VFS_WRITABLE=1 - make kernel filesystems writeable - needed for docker in a container
# SYSTEMD_SECCOMP=0 - disable capability checks
# SYSTEMD_NSPAWN_USE_CGNS=0 - https://wiki.archlinux.org/title/systemd-nspawn#Run_docker_in_systemd-nspawn
SYSTEMD_SECCOMP=0 systemd-nspawn -D "$MOUNT_POINT" --bind=/mnt/wsl:/mnt/wsl --bind=/:/mnt/host --bind-ro=/tmp/.X11-unix --capability=all "$@"

# --resolv-conf=bind-host - binds too early before the file is populated by network manager
EOF
chmod a+x /home/dariusza/mount-wsl2.sh
```
- set up docker container or service (Arch, replace Arch with Artix for Artix)
    - follow steps from [wsl2_docker](../windows/wsl2_docker.md) on the host if you want to run docker natively on the host
    - or setup the service below to mount a wsl2 container instead (todo: not everything works in a container)
```
cat << 'EOF' | sudo tee /etc/systemd/system/mount-wsl2-docker.service > /dev/null
[Unit]
Description=Wsl2 Docker mount service
RequiresMountsFor=/home/dariusza/wsl2-ntfs /tmp
Requires=systemd-modules-load.service
After=systemd-modules-load.service

[Service]
Environment="SYSTEMD_NSPAWN_USE_CGNS=0"
Environment="SYSTEMD_NSPAWN_API_VFS_WRITABLE=1"
Type=notify
ExecStart=/home/dariusza/mount-wsl2.sh "/home/dariusza/wsl2-ntfs/DockerArch/ext4.vhdx" "/home/dariusza/wsl2-docker-vhd/" "/dev/nbd0" --bind=/sys/fs/cgroup --boot # Artix: "/root/startdocker_init" Arch: --boot
Restart=no

[Install]
WantedBy=local-fs.target
EOF

# enable autostart
sudo systemctl enable mount-wsl2-docker.service
```
- add wsl container mount service (Arch, replace Arch with Artix for Artix)
```
# this sets up docker based on the DockerArch/DockerArtix container
# use wsl2_docker.md service instead if you want a to run on the host
WSL_INIT_SCRIPT=--boot # Artix: "/home/dariusza/bin/init" Arch: --boot
DOCKER_FS_PATH=/ # Native host: / Docker distro: /home/dariusza/wsl2-docker-vhd/
WSL_DISTRO_PATH=/home/dariusza/wsl2-ntfs/Arch/ext4.vhdx # Artix: 

cat << EOF | sudo tee /etc/systemd/system/mount-wsl2.service > /dev/null
[Unit]
Description=Wsl2 mount service
RequiresMountsFor=/home/dariusza/wsl2-ntfs /tmp
Requires=systemd-modules-load.service
After=systemd-modules-load.service mount-wsl2-docker.service

[Service]
Type=notify
ExecStart=/home/dariusza/mount-wsl2.sh "${WSL_DISTRO_PATH}" "/home/dariusza/wsl2-vhd/" "/dev/nbd1" --bind=/dev/dri --bind=${DOCKER_FS_PATH}home/dariusza/docker-bin:/mnt/wsl/docker-linux-wsl/bin --bind=${DOCKER_FS_PATH}:/mnt/wsl/docker-linux-wsl/root ${WSL_INIT_SCRIPT}
Restart=no

[Install]
WantedBy=local-fs.target
EOF

# enable autostart
sudo systemctl enable mount-wsl2.service
```
- set up a login shell script (Artix)
```
# use nsenter instead of machinectl/systemd-run becuase they require systemd in the container
# nsenter needs to be run as root
# use setuid to bypass permissions, bash ignores suid, so use an executable instead
#
# nsenter --target=$(machinectl show --property Leader wsl2-vhd | sed "s/^Leader=//") -a su dariusza
DOCKER_SUFFIX="" # DOCKER_SUFFIX="-docker" for the docker container
cat << EOF | gcc -o /home/dariusza/bash-wsl2${DOCKER_SUFFIX}.sh -xc -
#include <unistd.h>
#include <stdio.h>

int main(int argc, char** argv) {
  FILE *output;
  char arg1[1024];

  output = popen("machinectl show --property Leader wsl2-vhd$DOCKER_SUFFIX | sed 's/^Leader=//' | sed 's/^/--target=/' | tr --delete '\n'", "r");
  if (output == NULL) {
    printf("Failed to run command\n" );
    return 1;
  }

  // read the output
  fgets(arg1, sizeof(arg1), output);

  return execlp("nsenter", "nsenter", arg1, "-a", "-S0", "su", "-", "dariusza", (char *) NULL);
}
EOF
sudo chown root /home/dariusza/bash-wsl2${DOCKER_SUFFIX}.sh
sudo chmod u+sw,a+rx /home/dariusza/bash-wsl2${DOCKER_SUFFIX}.sh
```
- set up a login shell script (Arch)
    - the one for artix will work, alternatively you can use `machinectl shell --uid 1000 wsl2-vhd`
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
# artix - only:
sudo cp /home/dariusza/wsl2-vhd/home/dariusza/bin/host.conf /etc/sysctl.d/60-wslhost.conf
sudo sysctl -p /etc/sysctl.d/60-wslhost.conf
mkdir /tmp/cores
```
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
- faster io on linux by moving the image out of vhdx into a real ext4 partition which can be simply mounted:
    - https://docs.microsoft.com/en-us/windows/wsl/wsl2-mount-disk (currently in insider only)
    - alternatively try to make ext4 partition be visible as a special vhdx file? by somehow implementing [device files](https://en.wikipedia.org/wiki/Device_file) on windows? possibly by implementing it in linux and loading wsl partition from wsl share?
      - could make a wrapper wsl2 distro that mounts the main one like on linux?
    - linux kernel should get the new ntfs driver realtively soon, hopefully that'd make things run faster?

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
