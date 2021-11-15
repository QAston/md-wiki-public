## kubernetes

### distributions/implementation

- k3s
- kinD
- minikube

### management tools

- ctlpl - yaml config for a cluster
    - https://github.com/tilt-dev/ctlptl#features
    - pacaur -S ctlptl-bin
    

### automation

- tilt - docker compose but for k8s - easy deploymetns and cleanup
    - pacaur -S tilt-bin?

### setup - docker-desktop (windows and wsl2)

1. setup [docker](./docker.md) 
2. enable k8s in docker-desktop for wsl
3. set up completion - add to .bashrc:
```
if command -v kubectl &> /dev/null; then
 cmd_file=$(mktemp)
 kubectl completion bash > "$cmd_file"
 source "$cmd_file"

 rm -f "$cmd_file"
fi
```

### setup - docker-linux-wsl (native linux docker/k8s)

* follow [wsl2_host_container](../windows/wsl2_host_container.md)
* install minikube in docker-arch
```
pacman -S minikube
```
* point kubectl in arch to docker-arch minikube (possibly needs redoing every kube startup)
```
cat /mnt/wsl/host-arch/root/home/dariusza/.kube/config | sed s@/home/dariusza@/mnt/wsl/host-arch/root/home/dariusza@ > /home/dariusza/.kube/config
```

#### minikube usage

- [pushing images](https://minikube.sigs.k8s.io/docs/handbook/pushing/)

## devcontainers in k8s - okteto

- swaps the deployed k8s image for a devcontainer, provides file sync to the container and a shell to run commands in the k8s container
- [container examples and source](https://github.com/okteto/devenv)
    - looks like the containers don't need ANY addtional setup (other than devtools so that you can actually run commands)
- [rust full project example](https://github.com/okteto/rust-getting-started)

### setup

```
pacaur -S okteto
okteto analytics --disable
```

### okteto usage

```
kubectl apply -f app.yml # deploy app
okteto init # create manifest
okteto up # deploy the image from the manifest to current kubectl connection
```
- files changed locally will be uploaded, unless added to .stignore
- ports defined in manifest will be forwarded to local machine
- see [okteto vscode extension](./vscode.md) to debug from inside a container instead of using port forwarding, as well as using container build env and others