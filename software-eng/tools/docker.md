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

arguments agains docker - not neccessarily good
<http://www.boycottdocker.org/>
