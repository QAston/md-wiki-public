## kubernetes

### distributions/implementation

- k3s
- kinD
- minikube

### management tools

- ctlpl - yaml config for a cluster
    - https://github.com/tilt-dev/ctlptl#features
    - pacaur -S ctlptl-bin
- kustomize
    - make configuration variants declaratively without templates
    - can refer to files in other git repositories!
    - https://www.youtube.com/watch?v=1fCAwFGX38U
- helm
    - package manager + horrible templating engine
    - `helm upgrade --install` - idempotent install command
    - `helm install -f <file>` - values file, individual values can be set using `--set`
    - `helm template` evaluate the template

### automation

- tilt - docker compose but for k8s - easy deploymetns and cleanup
    - pacaur -S tilt-bin?
    - has support for nix: https://github.com/tilt-dev/tilt-extensions/tree/master/nix
    - extensions: https://github.com/tilt-dev/tilt-extensions
    - has suport for kustomize
    - has some live-reload capability: https://docs.tilt.dev/tutorial/5-live-update.html

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

- [nginx ingress](https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/)
- [haproxy ingress](https://haproxy-ingress.github.io/docs/getting-started/)
- [pushing images](https://minikube.sigs.k8s.io/docs/handbook/pushing/)
```
minikube start --driver=none # run minikube on host system and use host docker daemon (requires linux, no hypervisor, ideally debian docker package)

minikube dashboard # start a web dashboard, very useful for investigating

minikube service hopper-auth --url # temporarily expose a port of an internal service

kubectl logs service/hopper-router # show logs from a service

kubectl get service hopper-router -o yaml # show object yml

kubectl apply -k path/to/dir/with/kustomization.yml # deploy manifests referenced/configured by kustomizaton.yml 

kubectl delete -k path/to/dir/with/kustomization.yml # undeploy

minikube delete # delete the cluster

minikube stop # stop the cluster

echo "http://$(minikube ip):$(kubectl get service -n ingress-nginx ingress-nginx-controller -o json | jq -r .spec.ports[0].nodePort)" # ingress url
```

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