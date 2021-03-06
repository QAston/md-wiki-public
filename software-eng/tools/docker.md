# docker

docker build - builds dockerfile and puts it in local registry
docker run - start docker instance
  * docker run -t -i dk.tech.cit.lastmile.com/cfc_classic/activitydaemon bin/bash - explore the instance
  * docker exec -t -i mycontainer /bin/bash go to existing container
  * docker logs mycontainer - show logs printed by docker commands

windows:
  * docker toolbox for windows
    * docker-machine - manages docker host images on various vms
      * docker-machine default ip
        * shows the ip which is visible as host from the vm
    * toolbox's start.sh
      * if vm doesn't already exist
        * deletes old docker vm
        * creates a new vm with a fresh image, name: "default"
        * runs docker-machine.exe create -d virtualbox "default"
      * if not running starts vm
        * using docker-machine start default 
        * yes | docker-machine regenerate-certs
      * sets env variables using  docker-machine env default
    * docker.exe needs environment vars to work
      * docker-machine env default --shell={bash,cmd} # prints env vars to set in the syntax of a given shell
    * won't work:
      * sharing network with host?
  * docker for windows
    * hyper-v based
      * host ip: 10.0.75.1
    * no config required, docker.exe just works
    * don't work at ocado:
      * connecting from container to host
      * sharing files
    * won't work:
      * sharing network with host
  * docker for windows wsl2 backend
    * <https://docs.docker.com/docker-for-windows/wsl/>
    * preferred method

### setup docker-desktop (windows and wsl2 docker desktop integration)

- docker-desktop setup points linux executables at windows docker implementation, and uses wsl mostly for file hosting, it doesn't use linux daemons
- install <https://docs.docker.com/docker-for-windows/wsl/>
- remove any docker client/daemon setup existing inside wsl distros you want to use docker-desktop in (if any)
- run the following to start docker wsl integration (add to wsl-init.bat script to run on startup)
```
@rem start docker cli, starting once again starts docker daemon, then you can exit
start "C:\Program Files\Docker\Docker\Docker Desktop.exe" "C:\Program Files\Docker\Docker\Docker Desktop.exe" && exit
```
- once started docker executables will be visible both in windows and wsl2
- docker-desktop add links to executables in wsl2 distros (enabled/disabled for each distro in settings -> wsl integration):
  - /usr/bin -> usr/sbin -> /bin (arch/artix default mount)
  - /usr/bin/{docker,docker-compose} ->  /mnt/wsl/docker-desktop/cli-tols/usr/bin
  - /usr/local/bin/kubectl -> /mnt/wsl/docker-desktop/cli-tools/usr/local/bin/kubectl
  - /home/dariusza/.config/docker - locally generated config files
  - these links aren't added if a file already exists in the distro with the same name, so integration might not work if there are conflicts, but at least shouldn't delete files
- source completion - add to `~/.bashrc`
```
if [ -f /usr/share/bash-completion/completions/docker ]; then
    source /usr/share/bash-completion/completions/docker
fi
```

#### setup docker-linux in wsl2

follow [wsl2_host_container](../windows/wsl2_host_container.md)

### alternatives

- podman is a drop-in daemon-less replacement for docker
 
