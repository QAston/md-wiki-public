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
    * can copy some files over for arch instead of building them locally, run after "publish the guest filesytem to the host"
```
# wsl host only
sudo cp -ar /mnt/wsl/arch/root/home/dariusza/.ssh  /home/dariusza/.ssh
sudo cp -ar /mnt/wsl/arch/root/home/dariusza/.gnupg  /home/dariusza/.gnupg
sudo cp /mnt/wsl/arch/root/etc/locale.conf  /etc/locale.conf
sudo cp /mnt/wsl/arch/root/etc/locale.gen  /etc/locale.gen
cd /home/dariusza
git clone git@github.com:QAston/wslconfig.git
ln -s /home/dariusza/wslconfig/home/bin/ bin
cd wslconfig
./install.bash wsl
```
2. Add a windows startup script (wsl host only)
* docker_arch.bat:
```
wsl.exe -d DockerArch -u root -e subsystemctl start
```
3. Resize the filesystem to be big enough for docker and caches

### configure the host system

1. setup sshd for logging in from the guest container
```
sudo pacman -S openssh
echo "AcceptEnv FOO WSL_DISTRO_NAME" | sudo tee -a /etc/ssh/sshd_config # wsl host only
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
mkdir -p /home/dariusza/docker-bin/
ln -s /mnt/wsl/host-arch/root/bin/docker /home/dariusza/docker-bin/docker
ln -s /mnt/wsl/host-arch/root/bin/kubectl /home/dariusza/docker-bin/kubectl
cat << 'EOF' | tee /home/dariusza/docker-bin/minikube > /dev/null
#!/bin/bash

export MINIKUBE_HOME=/mnt/wsl/host-arch/root/home/dariusza/.minikube/
exec /mnt/wsl/host-arch/root/usr/sbin/minikube "$@"
EOF
chmod a+x /home/dariusza/docker-bin/minikube
```

5. set up mounts for usage from the guest system
```
cat << 'EOF' | sudo tee /etc/systemd/system/mnt-wsl-host\\x2darch-root.mount > /dev/null
[Unit]
Description=Bindmount for /mnt/wsl/host-arch/root to host system

[Mount]
What=/
Where=/mnt/wsl/host-arch/root
Type=none
Options=bind,uid=1000,gid=1000

[Install]
WantedBy=local-fs.target
EOF
sudo systemctl enable mnt-wsl-host\\x2darch-root.mount
sudo systemctl start mnt-wsl-host\\x2darch-root.mount

cat << 'EOF' | sudo tee /etc/systemd/system/mnt-wsl-host\\x2darch-bin.mount > /dev/null
[Unit]
Description=Bindmount for /mnt/wsl/host-arch/bin to host system

[Mount]
What=/home/dariusza/docker-bin
Where=/mnt/wsl/host-arch/bin
Type=none
Options=bind,uid=1000,gid=1500

[Install]
WantedBy=local-fs.target
EOF
sudo systemctl enable mnt-wsl-host\\x2darch-bin.mount
sudo systemctl start mnt-wsl-host\\x2darch-bin.mount
```
6. set up and configure services
```
# create an override that configures socket location and group
sudo mkdir -p /etc/systemd/system/docker.socket.d/
cat << 'EOF' | sudo tee /etc/systemd/system/docker.socket.d/override.conf > /dev/null
[Socket]
ListenStream=/mnt/wsl/host-arch/docker/docker.sock
SocketGroup=docker-linux
SocketUser=dariusza
DirectoryMode=0775
Writable=true
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
sudo systemctl enable docker.socket
sudo systemctl start docker.service
sudo systemctl start docker.socket
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
8. Set up links to the guest filesystem so that docker running on host can still see them for bindmount purposes
```
# use links instead of mount to not introduce an order dependency on guest
cd /home/dariusza/
ln -s -T /mnt/wsl/arch/root/home/dariusza/workspace workspace
```
9. Set up host directories to mount
- setup [nix](../tools/nix.md)
```
mkdir -p /home/dariusza/.cargo
sudo chown -R dariusza /home/dariusza/.cargo
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

[Mount]
What=/mnt/wsl/host-arch/root/nix/
Where=/nix
Type=none
Options=bind,uid=1000,gid=1000

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

[Mount]
What=/mnt/wsl/host-arch/root/home/dariusza/.cargo
Where=/home/dariusza/.cargo
Type=none
Options=bind,uid=1000,gid=1000

[Install]
WantedBy=local-fs.target
EOF
sudo systemctl enable home-dariusza-.cargo.mount
sudo systemctl start home-dariusza-.cargo.mount
```
3. publish the guest filesytem to the host
```
cat << 'EOF' | sudo tee /etc/systemd/system/mnt-wsl-arch-root.mount > /dev/null
[Unit]
Description=Bindmount for /mnt/wsl/arch/root to host system

[Mount]
What=/
Where=/mnt/wsl/arch/root
Type=none
Options=bind,uid=1000,gid=1000

[Install]
WantedBy=local-fs.target
EOF
sudo systemctl enable mnt-wsl-arch-root.mount
sudo systemctl start mnt-wsl-arch-root.mount
```
4. add key to the host
```
cat ~/.ssh/id_rsa.pub | /mnt/wsl/host-arch/root/home/dariusza/.ssh/authorized_keys
```

#### configure docker host integration on the guest system

1. set up users
```
sudo groupadd -g 1500 docker-linux
sudo gpasswd -a dariusza docker-linux
```
2. disable potentially conflicting packages on host distros by adding to IgnorePkg pacman.conf:
  * docker kubectl
3. set up the docker socket on the client system (some applicatons are bugged with DOCKER_HOST)
```
# arch - add a service to configure the socket
cat << 'EOF' | sudo tee /root/docker_client_socket.sh > /dev/null
#!/bin/bash
ln -s /mnt/wsl/host-arch/docker/docker.sock /var/run/docker.sock 
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
if [ -e /mnt/wsl/host-arch/docker/docker.sock ]; then
  export DOCKER_HOST="unix:///mnt/wsl/host-arch/docker/docker.sock"
fi
```
4. update .bash_profile
```
if [ -d /mnt/wsl/host-arch/bin ]; then 
   export PATH="$PATH:/mnt/wsl/host-arch/bin" # this is exported bin directory, not entire root/bin directory
fi 
```
6. update .bashrc
```
if [ -f  /mnt/wsl/host-arch/root/usr/share/bash-completion/completions/docker ]; then
  source /mnt/wsl/host-arch/root/usr/share/bash-completion/completions/docker
fi
if [ -f  /mnt/wsl/host-arch/root/usr/share/bash-completion/completions/minikube ]; then
  source /mnt/wsl/host-arch/root/usr/share/bash-completion/completions/minikube
fi
```

#### switching between integration with docker-host container and docker-for-windows

1. switching from docker-desktop to minikube requires:
```
rm -rf ~/.kube
rm -rf ~/.docker
sudo systemctl enable/disable docker-client-socket.service
```

#### mounting minikube in the guest container

- todo: it's possible to make minikube executable in the guest container talk to the host container (see <https://github.com/microsoft/vscode-dev-containers/blob/main/containers/kubernetes-helm/.devcontainer/copy-kube-config.sh>)
