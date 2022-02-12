## kubernetes

### distributions/implementation

- k3s
- kinD
- minikube

### management tools

- ctlpl - yaml config for a cluster - can be used to install minikube
    - https://github.com/tilt-dev/ctlptl#features
    - pacaur -S ctlptl-bin
- kustomize
    - make configuration variants declaratively without templates
    - can refer to files in other git repositories!
    - https://www.youtube.com/watch?v=1fCAwFGX38U
    - filtering out: `kustomize build . | kubectl apply -l name=<name> -f -`
- helm
    - package manager + horrible templating engine
    - `helm upgrade --install` - idempotent install command
    - `helm install -f <file>` - values file, individual values can be set using `--set`
    - `helm template` evaluate the template
- k9s - curses ui for kubectl

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

- minikube needs `--force-systemd` flag when running inside wsl2 for resource limits to work
- minikube can be run from a container with docker-from-docker when the container is started using `--network=host`, alternatively docker-in-docker can be used

##### ingress

- [nginx ingress](https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/)
- [haproxy ingress](https://haproxy-ingress.github.io/docs/getting-started/)
- [pushing images](https://minikube.sigs.k8s.io/docs/handbook/pushing/)
- the easiest method is just adding `--addons=ingress` flag to minikube startup

##### volumes on host system

- making k8s volumes that use host system direcories allows sharing directories between host system and containers, which is useful for development for example inside okteto

```bash
# start with a host mount
minikube start --mount=true --mount-string="/:/host" # mount entire host at once, using minikube mount to mount directories separately

# an example resource creation function
function host_volume {
  MINIKUBE_PATH=$2
  VOLUME_NAME_PREFIX=$1

  if ! kubectl get pv ${VOLUME_NAME_PREFIX}-pv >/dev/null 2>&1; then
  cat << EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: ${VOLUME_NAME_PREFIX}-pv
spec:
  storageClassName: standard
  accessModes:
    - ReadWriteMany
  capacity:
    storage: 500Gi
  hostPath:
    path: ${MINIKUBE_PATH}
EOF
fi
  if ! kubectl get pvc ${VOLUME_NAME_PREFIX}-pvc >/dev/null 2>&1; then
  cat << EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ${VOLUME_NAME_PREFIX}-pvc
spec:
  accessModes:
    - ReadWriteMany
  volumeMode: Filesystem
  resources:
    requests:
      storage: 500Gi
  volumeName: ${VOLUME_NAME_PREFIX}-pv
EOF
  fi
  echo "started external volume: ${VOLUME_NAME_PREFIX}-pvc"
}

host_volume "nix" "/host/nix" # mount an example directory for sharing with host or caching between minikube instances
```

##### commands

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

### setup

```
pacaur -S okteto
okteto analytics --disable
```

### example config

```yml
name: deployment-name-to-override
image: devcontainer-image
workdir: /home/user/app # workdir for command
command: ["bash", "--login"] # the command executed when running okteto up (not when connecting to the ssh server or using okteto exec or using remote k8s extension)
sync:
 - .:/home/user/app # files to synchronize; todo could use host volumes instead?
forward:
  - 8080:8080 # ports to forward to localhost
resources: # resources to change to have enough build fast
  limits:
    cpu: "3"
    memory: "8Gi"
securityContext: # set the same as user inside the container
  runAsUser: 1000
  runAsGroup: 1000
  fsGroup: 1000
  capabilities:
    add:
    - SYS_PTRACE # enable debugging
imagePullPolicy: Never # disable pulls if you want to use a locally built devcontainer
persistentVolume:
  enabled: true
volumes:  # make these directories persist between okteto up/down runs
  - /nix
# alternatively use k8s host volumes instead of regular volumes for sharing or caching between minikube instances
#externalVolumes:
#  - nix-pvc:/nix
```

### how it works

- swaps the deployed k8s image for a devcontainer, provides file sync to the container and a shell to run commands in the k8s container
- okteto sets up a custom ssh server called [remote](https://github.com/okteto/remote)
    - searches for first in the path bash/sh <https://github.com/okteto/remote/blob/main/pkg/os/os.go>
    - ssh command execution <https://github.com/okteto/remote/blob/99de42c0414824ace9f8f93b99ac7dd35f212fb2/pkg/ssh/ssh.go#L325>
- each connecting ssh process spawns bash (or sh if bash not found)
    - the spawned bash processes inherit the okteto remote's environment - that's how .rc files are pulled in
    - because it executes bash with no flags on ssh connection, setting env `BASH_ENV=/path.sh` can be used as a .rc script that's executed for every ssh connection
    - okteto's `command` doesn't have ANY influence on remote's environment (and also, it's probably started after the ssh server)
    - there are okteto-specific env variables set in the container:
      - OKTETO_NAME, OKTETO_NAMESPACE, as well as kubernetes specific variables
    - options for replacing the shell environment (for nix-shell compatibility for example)
      - BASH_ENV override - add condition to current BASH_ENV script (chaning BASH_ENV doesn't work because for some reason bash sees old value of the variable, but adding a new variable works just fine)
```bash
# load nix shell if specifically requested using BASH_ENV_LOAD_NIX_SHELL and the shell isn't used to launch another command
if [ -n $BASH_ENV_LOAD_NIX_SHELL ] && [ -z $IN_NIX_SHELL ] ; then
  unset BASH_ENV_LOAD_NIX_SHELL
  if [ -z $BASH_EXECUTION_STRING ] ; then
    exec cached-nix-shell /home/user/app/shell.nix
  else
    exec cached-nix-shell /home/user/app/shell.nix --command "$BASH_EXECUTION_STRING"
  fi
fi
```
  - wrapper script for okteto's `./server.sh` -   
  - somehow pointing okteto at a different bash (it searches in PATH, so modifying PATH would work)
- okteto up creates a standard ssh client configuration pointing at the server
- okteto forwards ports using k8s port forwarding, which are then exposed to ssh

### okteto usage

```
kubectl apply -f app.yml # deploy app as normal
okteto init # create manifest
okteto up # deploy the image from the manifest to current kubectl connection; then start ssh server, synchronize the files and finally run the given `command`
# exit in the okteto up shell: shuts down the container
okteto down # delete the container, bring back the old manifest
okteto down -v # delete the image including the persistent volumes - full cleanup
```
- files changed locally will be uploaded, unless added to .stignore
    - okteto up -l debug to see upload debugging
    - upload issues are usually caused by weird/big directories, remove unnecessary dirs and try again
- ports defined in manifest will be forwarded to local machine
- see [okteto in vscode](./vscode.md) to debug from inside a container instead of using port forwarding, as well as using container build env and others
- [container examples and source](https://github.com/okteto/devenv)
    - looks like the containers don't need ANY addtional setup (other than devtools so that you can actually run commands)
- [rust full project example](https://github.com/okteto/rust-getting-started)

## tilt

- tilt - docker compose but for k8s - easy deploymetns and cleanup
    - `pacaur -S tilt-bin`?
    - has support for nix: <https://github.com/tilt-dev/tilt-extensions/tree/master/nix>
    - extensions: <https://github.com/tilt-dev/tilt-extensions>
    - has suport for kustomize
    - has some live-reload capability: <https://docs.tilt.dev/tutorial/5-live-update.html>
