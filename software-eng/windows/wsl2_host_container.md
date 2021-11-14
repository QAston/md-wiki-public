## wsl2 host container

- a separate wsl2 container which provides heavy "host" functionality to make the main container an easily portable "guest"
  - the guest is then portable between windows and [native wsl host](../linux/native_wsl_host.md), with the container not seeing any difference
- provided functionality:
  - docker and related daemons
    - [inspired by](https://dev.to/bowmanjd/install-docker-on-windows-wsl-without-docker-desktop-34m9)
    - this means the main container can switch between using this and docker-desktop
    - sadly, this setup means that binding docker volume will bind on the drive of the docker container, not the container that just uses the socket
  - lots of space for various caches (nix, rust, docker, etc)
- the wsl2 host container itself could theoretically also be run inside [native wsl host](../linux/native_wsl_host.md) to share the caches between hosts
    - however, it's not a good idea becaus there are multiple issues (for example running docker in systemd-nspawn) which would undermine the purpose of running natively on a real linux system 

### setup wsl2 host container

1. create a new [wsl2 arch container](./wsl2_arch.md) to run the host
    * before installing, rename Arch.exe to DockerArch.exe to install a new distro instead of override and existing one
    * can copy some files over for arch instead of building them locally
```
cp -r /mnt/wsl/Arch/home/dariusza/portable/ .
sudo cp -afr /mnt/wsl/Arch/bin/subsystemctl* /usr/bin
sudo sed -i 's|:/bin/bash|:/bin/subsystemctl-bash|g' /etc/passwd
sudo cp /mnt/wsl/Arch/etc/locale.conf  /etc/locale.conf
sudo cp /mnt/wsl/Arch/etc/locale.gen  /etc/locale.gen
```
2. Add a windows startup script (wsl containers)
* docker_arch.bat:
```
wsl.exe -d DockerArch -u root -e subsystemctl start
```
3. Resize the filesystem to be big enough for docker and caches

### configure the host system

1. setup sshd for logging in from the guest container
```
sudo pacman -S openssh
sudo systemctl enable sshd
sudo systemctl start sshd
```
2. setup docker
```
sudo pacman -S docker minikube kubectl
```
3. set up users
```
sudo groupadd -g 1500 docker-linux
sudo gpasswd -a dariusza docker-linux
```
4. set up ~/docker-bin directory for integration
```
sudo mkdir -p /home/dariusza/docker-bin/
sudo ln -s /mnt/wsl/docker-linux-wsl/root/bin/docker /home/dariusza/docker-bin/docker
sudo ln -s /mnt/wsl/docker-linux-wsl/root/bin/kubectl /home/dariusza/docker-bin/kubectl
```
5. add an init script
```
# create a service to handle mounts
cat << 'EOF' | sudo tee /etc/systemd/system/mount-wsl2-docker.service > /dev/null
[Unit]
Description=Wsl2 mount service for docker containers
DefaultDependencies=no
After=sysinit.target system.slice

[Service]
ExecStart=/root/wsl2-docker-mounts.sh
Restart=no
Type=oneshot
EOF
# script for the service
cat << 'EOF' | sudo tee /root/wsl2-docker-mounts.sh > /dev/null
#!/bin/bash
mkdir -pm o=rx,ug=rwx /mnt/wsl/docker-linux-wsl/docker
mkdir -pm o=rx,ug=rwx /mnt/wsl/docker-linux-wsl/bin
mkdir -pm o=rx,ug=rwx /mnt/wsl/docker-linux-wsl/root
chgrp docker-linux /mnt/wsl/docker-linux-wsl /mnt/wsl/docker-linux-wsl/docker /mnt/wsl/docker-linux-wsl/bin

mount --bind /home/dariusza/docker-bin "/mnt/wsl/docker-linux-wsl/bin"
mount --bind / "/mnt/wsl/docker-linux-wsl/root"
EOF
sudo chmod a+x /root/wsl2-docker-mounts.sh

# run the service
sudo systemctl enable mount-wsl2-docker.service
```
6. set up and configure services
```
# create an override that configures socket location and group
sudo mkdir -p /etc/systemd/system/docker.socket.d/
cat << 'EOF' | sudo tee /etc/systemd/system/docker.socket.d/override.conf > /dev/null
[Unit]
Requires=mount-wsl2-docker.service
After=mount-wsl2-docker.service
[Socket]
ListenStream=/mnt/wsl/docker-linux-wsl/docker/docker.sock
SocketGroup=docker-linux
EOF
# create an override that configures the docker group
sudo mkdir -p /etc/systemd/system/docker.service.d/
cat << 'EOF' | sudo tee /etc/systemd/system/docker.service.d/override.conf > /dev/null
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// -G docker-linux
EOF
sudo systemctl daemon-reload
sudo systemctl enable docker.service
sudo systemctl start docker.service
```
7. Configure docker to use windows dns for dns in the default network (wsl host container only)
```
# ip addresses taken from ipconfig /all
cat << 'EOF' | sudo tee /etc/docker/daemon.json > /dev/null
{
    "dns": ["194.168.4.100", "194.168.8.100"]
}
EOF
```
8. Set up a host workspace directory, so that bindmounts work
```
mkdir /home/dariusza/host-workspace
```

### configure host integration in the guest container

1.  set up sharing /nix directory with the host (so that's it's accessible as a docker bindmount)
```
sudo rm -rf /nix
sudo mkdir /nix
sudo chown dariusza /nix
sudo chgrp dariusza /nix
# create a service
cat << 'EOF' | sudo tee /etc/systemd/system/nix.mount > /dev/null
[Unit]
Description=Link for the docker.socket from /mnt/wsl
After=local-fs.target umount.target

[Mount]
What=/mnt/wsl/docker-linux-wsl/root/nix/
Where=/nix
Type=none
Options=bind

[Install]
WantedBy=local-fs.target
EOF
sudo systemctl enable nix.mount
sudo systemctl start nix.mount

# might need to add additonal users
cd /nix/var/nix/profiles/per-user
ln -s dariusza user
```
2. set up sharing rust build cache with the host (so that's it's accessible as a docker bindmount)
```
# create on host system or copy from guess system
mkdir /home/dariusza/.cargo

# create a service on guest system
cat << 'EOF' | sudo tee /etc/systemd/system/home-dariusza-.cargo.mount > /dev/null
[Unit]
Description=Bindmount for /home/dariusza/.cargo to host system
After=local-fs.target umount.target

[Mount]
What=/mnt/wsl/docker-linux-wsl/root/home/dariusza/.cargo
Where=/home/dariusza/.cargo
Type=none
Options=bind

[Install]
WantedBy=local-fs.target
EOF
sudo systemctl enable home-dariusza-.cargo.mount
sudo systemctl start home-dariusza-.cargo.mount
```
3. set up shared workspace with the host (so that's it's accessible as a docker bindmount)
```
# create on host system or copy from guess system
mkdir /home/dariusza/host-workspace

# create a service on guest system
cat << 'EOF' | sudo tee /etc/systemd/system/home-dariusza-host\\x2dworkspace.mount > /dev/null
[Unit]
Description=Bindmount for /home/dariusza/host-workspace to host system
After=local-fs.target umount.target

[Mount]
What=/mnt/wsl/docker-linux-wsl/root/home/dariusza/host-workspace
Where=/home/dariusza/host-workspace
Type=none
Options=bind

[Install]
WantedBy=local-fs.target
EOF
sudo systemctl enable home-dariusza-host\\x2dworkspace.mount
sudo systemctl start home-dariusza-host\\x2dworkspace.mount
```

#### configure docker host integration - need to also do these on host if you want to use docker from host

1. set up users
```
sudo groupadd -g 1500 docker-linux
sudo gpasswd -a dariusza docker-linux
```
2. disable potentially conflicting packages on host distros by adding to IgnorePkg pacman.conf:
  * docker kubectl
3. set up the docker socket on the client system (some applicatons are bugged with doc)
```
# arch - add a service to configure the socket
cat << 'EOF' | sudo tee /root/docker_client_socket.sh > /dev/null
#!/bin/bash
ln -s /mnt/wsl/docker-linux-wsl/docker/docker.sock /var/run/docker.sock 
chmod g+rwx /var/run/docker.sock
chgrp docker-linux /var/run/docker.sock
EOF
sudo chmod a+x /root/docker_client_socket.sh
# create a service
cat << 'EOF' | sudo tee /etc/systemd/system/docker-client-socket.service > /dev/null
[Unit]
Description=Link for the docker.socket from /mnt/wsl
DefaultDependencies=no
After=sysinit.target system.slice

[Service]
ExecStart=/root/docker_client_socket.sh
Restart=no
Type=oneshot

[Install]
WantedBy=sockets.target
EOF
sudo systemctl enable docker-client-socket.service
sudo systemctl start docker-client-socket.service
```
* alternatively - add the following to .bashrc - don't do this as some applications don't work with DOCKER_HOST set, like tilt + minikube
```
if [ -e /mnt/wsl/docker-linux-wsl/docker/docker.sock ]; then
  export DOCKER_HOST="unix:///mnt/wsl/docker-linux-wsl/docker/docker.sock"
fi
```
4. update .bash_profile
```
if [ -d /mnt/wsl/docker-linux-wsl/bin ]; then 
PATH="$PATH:/mnt/wsl/docker-linux-wsl/bin"
fi
```
5. update .bashrc
```
if [ -f  /mnt/wsl/docker-linux-wsl/root/usr/share/bash-completion/completions/docker ]; then
  source /mnt/wsl/docker-linux-wsl/root/usr/share/bash-completion/completions/docker
fi
```
6. add key to the host
```
cat ~/.ssh/id_rsa.pub | /mnt/wsl/docker-linux-wsl/root/home/dariusza/.ssh/authorized_keys
```

#### switching between integration with docker-host container and docker-for-windows

1. switching from docker-desktop to minikube requires:
```
rm -rf ~/.kube
rm -rf ~/.docker
sudo systemctl enable/disable docker-client-socket.service
```
