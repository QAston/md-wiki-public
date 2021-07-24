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

### setup docker-linux-wsl

* follow [wsl2_docker](../windows/wsl2_docker.md)
* install minikube
* todo: find a way to connect remotely to minikube on another container (minikube --apiserver?)