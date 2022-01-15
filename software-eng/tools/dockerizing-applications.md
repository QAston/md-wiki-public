Dockerizing Applications
================

In order to properly dockerize an application you need to be aware of how exactly the application uses its resources and IO.

## Configuring applications

To properly configure your application in docker you first need to be aware of the resources used by your application and you need to know how to configure the resource usage.

### Filesystem

Docker uses a [CopyOnWrite filesystem](https://docs.docker.com/storage/storagedriver/) by default, which has the following effects:
* All i/o operations go through a layer of indirection (COW driver), which incurs a (small) performance penalty on every operation
* The first modification to an existing file is slow because the file has to be copied (the larger the file the longer it takes)
* Changes to the files remain there as long as the exact same container is being restarted, but are lost after container is removed or recreated (no persistence)
* The more layers the more inefficient the filesystem is.

Docker provides a [volume/bindmount mechanism](https://docs.docker.com/storage) for applications which need persistence or fast i/o. Make sure to document in container README which directories need to be mounted as a volume/binmount for persistence and optimal performance. Also, if you're using bindmount, make sure that the filesystem user id of the user in the container matches the container's userid for optimal bindmount - you might need to rebuild the image with appropriate uid.

To configure the volumes you can use:
 * `-v` flag [in docker](https://docs.docker.com/storage/volumes/#choose-the--v-or---mount-flag)
 * `volumes:` [declarationin in docker-compose](https://docs.docker.com/compose/compose-file/#volumes)

### Network

#### Docker networks

Docker has different networking modes:
- default `bridge` network - containers can see each other through IPs but not through dns names
    - /etc/resolv.conf and other dns config is inherited from host
- `docker network create -d bridge name` `--network=name` - named bridge network, containers can see each other through container names if they're part of the same network
- `--network=host` - share network interfaces with the host, fastest network io
- `--network=container-id|name` - share network interfaces with a specific container
- `--network=none` - no network access

Communicating with the host:
- [ ] on linux add: `--add-host=host.docker.internal:host-gateway`, then use host.docker.internal dns name to connect to host
    - on windows/mac don't need to add the flag, the host dns is added automatically
- use -p/-P flags to expose ports to host
Communicating with other containers:
- all containers that are share the network are accessible (docker network connect to add a container to multiple networks)
- containers within network can connect to any port without the need to publish/expose

#### Network references

To allow your containers to be deployed in any network configuration (docker-compose, service-inventory, mixed environments) you need to make sure that all variable network references used (URLs, IPs, DNSes, ports, etc) are configurable, for example by using env variables.
The configuration should be done through [environment variables](#dockerizing-applications_configuring-applications_environment-configuration) and follow the naming conventions:
* URLs should be configurable using `<NAME>_URL` env variable convention:
* Hostnames(DNS names/IPs) and ports should be configurable using `<NAME>_HOST` and `<NAME>_PORT` env variable convention
* References (URLs/Hostnames/ports) which are sent to clients outside of the docker network should be configurable using env variables with `PUBLIC_` prefix. This reference can then be configured to the NAT/ingress adress or the mapped port and allow sending the clients links that work from their network.
    * Example: 2 reporting services are in a single compose network. If one service wants to query another it uses `<SERVICENAME>_URL`. If the service wants to send a link to another service to the client browser, it sends the `PUBLIC_<SERVICENAME>_URL` link, as browsers are never in the docker network. Alternatively a setup with a reverse-proxy can be used (see http/https reverse proxy headers: https://www.nginx.com/resources/wiki/start/topics/examples/forwarded/ )
    * This solution has a limitation - a single NAT/ingress/port-mapping has to work for all clients. To work around that you need to know which client is coming from which network and send appropriate reference accordingly.
* Back-references to the container itself are a special case:
    * Back-references used privately by the container itself should be done as usual using `localhost` and [exposed port](#dockerizing-applications_writing-dockerfiles_container-processes_entrypoint-declaration) numbers, no need for configuration
    * Back-references sent to other containers within the same docker network should use container hostname (in java `InetAddress.getLocalHost().getHostName()`). This value is configured automatically by most docker environments (`--hostname`).
    * Back-references sent to **HTTP/HTTPS** clients should use a reverse proxy which sets forwarding headers, the application needs to be configured to handle the headers, see: https://www.nginx.com/resources/wiki/start/topics/examples/forwarded/
    * Back-references sent to other clients outside the docker network should be configurable using `PUBLIC_HOSTNAME` (also `PUBLIC_<NAME>_PORT` or `PUBLIC_<NAME>_URL` if needed)

### Docker daemon configuration

todo: elaborate on considerations for each fo these:
* standard docker - docker daemon on local machine
* remote docker - docker accessed through a docker socket
* docker-from-docker - docker daemon accessed from inside of a docker container through a docker docket
* docker-in-docker - docker daemon spawned inside a container

## Writing dockerfiles

Next step after knowing how your application works is creating a Dockerfile which is executed by `docker build` to create the image.

Typical Dockerfiles follow this structure:

1. [Base image declaration](#dockerizing-applications_writing-dockerfiles_base-image)
2. [Populating the image's filesystem](#dockerizing-applications_writing-dockerfiles_filesystem)
3. [Configuration of the main process](#dockerizing-applications_writing-dockerfiles_container-processes)

### Base image

There are several interesting options for chosing base images:
* [distroless images - no distro at all](https://github.com/GoogleContainerTools/distroless)
* [alpine images - very small distro using musl instead of glibc](https://hub.docker.com/_/alpine)
* [vscode devcontainers - dev environments usable from vscode](https://github.com/microsoft/vscode-dev-containers)
* [nix containers - only include what you use](https://nix.dev/tutorials/building-and-running-docker-images)

Make sure to declare an exact base image version instead of latest, so that the container build is deterministic

### Filesystem

Use [dockerfile instructions](#dockerizing-applications_writing-dockerfiles_filesystem_dockerfile-instructions) to populate the filesystem. Populating the filesystem usually involves:

1. Installing any dependencies used by your application, some of that will be done by the base image.
2. Installing your application (usually by copying the jars/unpacking the tarball into the right directory)
3. Setting filesystem `chown`/`chmod` access rights, so that [user running the processes](#dockerizing-applications_writing-dockerfiles_container-processes_user-running-the-processes) can access the needed files

#### Dockerfile instructions

A short summary of the filesystem instructions (see also [Dockerfile reference](https://docs.docker.com/engine/reference/builder/#run)):
* RUN - executes a command on top of the image
    * `RUN <somecommand> <arg1> <arg2>` will run in /bin/sh (can be changed using SHELL instruction)
    * `RUN [<executable>, <arg1>, <arg2>]` will run the executable directly
    * **Every docker instruction which makes changes to the filesystem  creates a separate layer in the [CopyOnWrite filesystem](https://docs.docker.com/storage/storagedriver/) containing a copy of all changed files (even if only metadata changed, and even if the files are intermediate files that'd be deleted in a next layer).** Chaining operations in RUN commands reduces the image size because it results in fewer, individually smaller, layers. Chaining looks like this:
```
RUN <command1> <arg1> && \
    <command2> <arg2>
```
* `WORKDIR <dirname>` - combines cd and mkdir, sets dir for RUN command, last WORKDIR will be the dir in which the ENTRYPOINT process will be run; initial workdir is `/` (root)
* `USER <uid>` - set the uid for the user as which RUN commands will execute, last USER directive will set as which user ENTRYPOINT process will be run; initial USER is 0 (root!)
* COPY/ADD - copy files into a docker image layer
    * COPY vs ADD - both work the same except COPY simply copies `<src>` to `<dest>`, but ADD  has special handling for some types of `<src>`:
        * if ADD `<src>` is an archive it'll will be unpacked as a directory, with owner being the owner defined in the archive (windows built archives set owner to root by default).
        * if ADD `<src>` is a url, the contents will be downloaded
    * 2 syntaxes:
        * `COPY <src>... <dest>` (can't have whitespaces in paths)
        * `COPY ["<src>",... "<dest>"]`
    * Copied directories and files preserve their attributes from the source filesystem; 
        * all directories that are newly created and not copied from the source filesystem are created as uid 0 (root!), [even if a USER directive is active](https://github.com/moby/moby/issues/11246)
        * passing a `--chown=<uid>:<gid>` flag will only affect the newly-created directories and files (and files from windows without owner), ownership of files copied/extracted is not affected
        * windows doesn't have unix file attributes, so [files copied from windows filesystems always have `755` permissions](https://github.com/moby/moby/pull/11395) and owner of root or `--chown` if set
        * if you're on windows and you're adding an executable file (for example bash script) to a git repository used to build a docker image you need to make sure you run `git update-index --chmod=+x filename.sh`, otherwise linux users (and CI) will not be able to build the image
    * `<dest>` and parent directories are always created if missing
    * relative `<dest>` paths are relative from last `WORKDIR` directive
    * `<dest>` are trailing slash sensitive:
        * if `<dest>` ends with `/` the `<dest>` will be a directory to which files are copied
        * otherwise there can only be 1 `<src>` and it's contents will be copied to `<dest> file
    * `<src>` can be [go file wildcards](http://golang.org/pkg/path/filepath#Match)
    * `<src>` paths are relative to the `<path>` passed to `docker build <path>` and can only refer to files inside `<path>`
    * if `<src>` is a directory the contents of the directory will be copied, but not the directory itself
* ARG/ENV - declaring and using variables:
    * Dockerfile instructions which do not delegate execution to external processes can refer to previously declared Dockerfile variables using following [syntax](https://docs.docker.com/engine/reference/builder/#environment-replacement):
        * `$VAR_NAME` or `${VAR_NAME}` - substitute with the value
        * `${VAR_NAME:-defaultvalue}` - if `$VAR_NAME` is set use it, otherwise substitute `defaultvalue`
        * `${VAR_NAME:+overwritevalue}` - if `$VAR_NAME` is set use `overwritevalue` otherwise substitute empty string
    * Dockerfiles allow declaring 2 kinds of variables:
        * `ARG VAR_NAME=<default-value>` - declares Dockerfile variables whose values can be set using `docker build`'s flags: `--build-arg NAME1=value1 --build-arg NAME2=value2`
        * `ENV VAR_NAME=<value>`- declares Dockerfile variables which (unlike ARGs) also function as environment variables, which means that external processes in instructions like `RUN` and `ENTRYPOINT` can access their value. The syntax for accessing the values of environment variables depends on the executed process, in bash you can use `${VAR_NAME}` syntax, in powershell you can use `$env:VAR_NAME` and in java you can use `System.getenv("VAR_NAME")`. The final value of ENV variables is persisted with the image and those environment variables remain available for docker container processes. You can change and create new environment variables by passing `--env VAR1=value1 --env VAR2=value2` flag to `docker create`/`docker run`.
    * To make the ENV variable configurable during build (using `docker build --build-arg=`)you can use a helper ARG variable: 
    ```
    ARG VAR_NAME=<default>
    ENV VAR_NAME=$VAR_NAME
    ```
* VOLUME - Declares an anonymous volume to be created. If any docker build steps change the data within the volume after it has been declared, those changes will be discarded - volume needs to be declared last. **Avoid using this declaration** and prefer declaring the volume in docker-compose/service-inventory/`docker-run` because:
    * The volume can be changed with `--volume` flag on container creation but it can’t be disabled if you don't want it (there’s no flag to do that)
    * VOLUMEs can't be modified in derived images, all modifications to the directories are silently discarded 
    * Deleting the container will not delete the volume, the volume needs to be deleted explicitly after the container is deleted (`docker volume prune -f`, `docker compose down -v`, `docker run --rm`)
    * Each new container with an anonymous volume gets one created from scratch, they’re not reused unless refered-to by name explicitly, which may result in lots of orphaned volumes
* ONBUILD `<NORMAL INSTRUCTION>` - run the instruction when the image is used as a base, not inherited transitively by grandchildren
* SHELL - override shell used for shell-form entrypoint definitions
* STOPSIGNAL - define signal sent to container when it's stopped, by default it's SIGTERM

### Env variables

For linux containers docker sets the following variables:
- HOME
- HOSTNAME
- PATH
- TERM

### Container processes

Docker containers run the following processes:
* main process which optionally spawns subprocesses
* [healthcheck](#dockerizing-applications_writing-dockerfiles_container-processes_configuring-healthcheck) (optional)
* additional processes started manually by users using `docker exec`

To properly configure the main process first you need to:

1. [learn how the main process works](#dockerizing-applications_writing-dockerfiles_container-processes_main-container-process)
2. [find a user to run the process and configure the filesystem](#dockerizing-applications_writing-dockerfiles_container-processes_user-running-the-processes)
3. [declare the entrypoint](#dockerizing-applications_writing-dockerfiles_container-processes_entrypoint-declaration)

#### Main container process

* The lifecycle of the main process is directly linked to the lifecycle of the container:
    *  `docker start`, `docker stop`, `docker kill` and similar commands are starting, stopping and sending signals directly to the main process
    *  Once the process stops the container is stopped.
    *  When container shutdown is requested (`docker stop`) the main process receives SIGTERM signal, which gives it 10 seconds (configurable using `--stop-timeout=` flag of `docker create`) to cleanup and shutdown, after which all alive processes are forcefully killed. Docker sends signals only to the main process, so if you've started any child processes that need cleanup (e.g to make a db commit, close tcp socket), make sure to forward SIGTERM to them or to manually trigger their cleanup or they'll be forcibly killed.
* The main process and it's container environment (shared by all container processes) can be configured/overridden by passing flags to `docker create`/`docker run` commands (or their equivalents). This configuration is guaranteed to be immutable (so you don't need to worry about environment variables changing) and if you want to change it you need to create a new container instead. Example configuration includes ([among others](https://docs.docker.com/v17.09/engine/reference/commandline/create/#options)):
    * `--env VAR1=value1 --env VAR2=value2` - environment variables
    * `--volume`/`--mount` - declare mountpoints for volumes
    * `--shm-size` - size of /dev/shm which allows using ram instead of filesystem
    * `-p=<hostport>:<containerport>` - publish a port to host
    * `--expose=<containerport>` - same as EXPOSE directive
    * `--hostname` - override hostname of the container
    * `--pid=` - override PID of the process(`1` by default)
    * `--user=<uid>[:<gid>]` - override USER running the process
    * `--init=true` - create an init process for managing children processes and forwarding signals to them by using the `docker-init` process found on host, typically [tini](https://github.com/krallin/tini)
* Dynamic (mutable) configuration of docker containers can be both set using `docker create` and altered using `docker update`, examples include ([among others](https://docs.docker.com/v17.09/engine/reference/commandline/update/#options)):
    * `--memory=` -  limits resident memory (physically in ram), unlimited by default
    * `--memory_swap=` - limits total of physical + swap
        * if `--memory` is unset the default is unlimited, otherwise it's set to 3x memory, so that you can use 2x memory as swap
        * swap needs to be available on the host system, otherwise this won't have any effect
    * `--cpu-shares` - relative cpu usage weight
    * `--cpus` - absolute cpu count (with fractions)

#### User running the processes

To be able to run your image on Nautilus, the container needs to be configured to run processes as a [non-root user](https://sre.pages.tech.lastmile.com/platform/topic/base-images.html#image-configuration).

1. Find the uid:gid of an appropriate user, `nobody` (on ubuntu/debian 65534, on centos 99) is usually the recommended user.  To get the uid:gid you can run `grep nodoby /etc/passwd` in the base image.
2. Setting filesystem `chown`/`chmod` access rights, so that user running the processes can access the needed files, for example:
```
RUN chmod -R <uid>:<gid> <list of files and directories>
```
3. If your application requires some of the root capabilities to run **(most applications don't and you can skip this step)** enable the required root capabilities (examples: `cap_net_bind_service` - opening ports < 1024, `cap_net_raw` - sending packets which aren't TCP/UDP):
    1. Find the `<real-path-to-java-executable-and-not-a-link>` (this method doesn't work with symlinks), you can use `whereis` to find the link and `ls -al` to find where the link leads 
    2. As root USER: `RUN setcap cap_net_bind_service,cap_net_raw=+epi <real-path-to-java-executable-and-not-a-link>`
    3. Find the `<real-path-to-jre/lib/jli-directory-and-not-a-link>` usually it's `<path-to-jre>/lib/amd64/jli`
    4. As root USER: `RUN echo '<path-to-jre>/lib/amd64/jli' > /etc/ld.so.conf.d/java.conf && ldconf`, otherwise you'll experience java-specific issue ["error while loading shared libraries"](https://bugs.openjdk.java.net/browse/JDK-7076745) when starting `java` as non-root user
4. Switch the user with `USER <uid>` for the [entrypoint declaration](#dockerizing-applications_writing-dockerfiles_container-processes_entrypoint-declaration).
5. Make sure that the [entrypoint script](#dockerizing-applications_writing-dockerfiles_container-processes_entrypoint-script) (if any) has the executable flag set (either by setting it in the repository `git update-index --chmod=+x docker/*.sh` or by setting it using `RUN chmod a+x ./docker-entrypoint.sh`.

#### Entrypoint declaration

The entrypoint declaration consists of the following clauses:
* USER - see [user running the process](#dockerizing-applications_writing-dockerfiles_container-processes_user-running-the-processes)
* WORKDIR - workdir the process will start with
* ENTRYPOINT/CMD - specifies the [main container process executable](#dockerizing-applications_writing-dockerfiles_container-processes_main-container-process)
    * usually a [./docker-entrypoint.sh script](#dockerizing-applications_writing-dockerfiles_container-processes_entrypoint-script), can be inlined into the ENTRYPOINT clause if small
    * Remember to use `["./docker-entrypoint.sh"]` (array form) or `ENTRYPOINT exec ./docker-entrypoint.sh` (shell form with exec), `ENTRYPOINT ./docker-entrypoint.sh` - shell form without exec will wrap the declared process in a shell process (even if that process is itself a shell process), which will prevent your application from receiving signals correctly (which is necessary for handling process cleanup)
    * Prefer using ENTRYPOINT over CMD for applications (see [entrypoint and cmd interaction](https://docs.docker.com/engine/reference/builder/#understand-how-cmd-and-entrypoint-interact)), CMD is meant for tools (where you need to change startup command)
    * you must specify either CMD or ENTRYPOINT, otherwise build will be rejected
    * if cmd is defined in base image, setting entrypoint will reset it to empty value
* EXPOSE - declare the ports your application is going to listen on (wcs port, jmx...)
    * Docker-compose will automatically expose those ports to other services in the compose network
    * `docker run -P` will expose these ports to the host operating system 
    * This declaration the recommended method because it's convenient and serves as documentation but it's not required as any ports can always be exposed later when container is being started by using `--expose=<containerport>`, `-p<hostport>:<containerport>` flag or equivalents.

Example entrypoint declaration (in Dockerfile):
```
# UID of the user with which the process will be run
USER 65534
# Working directory in which the process will run
WORKDIR /app
# The main container process executable
ENTRYPOINT ["./docker-entrypoint.sh"]
# Ports exposed by the process
EXPOSE 9002
```

#### Configuring healthcheck

You can optionally declare a healthcheck helper process to be periodically run to publish the status of the main process:
* this declaration is optional, as standalone docker uses the main process to verify that container is running. Status of HEALTHCHECK is displayed in docker ps, but ignored by docker process.
* Nautilus (service-inventory) [has it's own healthcheck declaration](https://sre.pages.tech.lastmile.com/platform/checks.html) which probably overrides the ones defined in Dockerfiles.
* [docker-compose v2.1](https://docs.docker.com/compose/compose-file/compose-file-v2/#depends_on) will delay startup of dependent containers until dependencies are started, if requested using `depends_on.condition: service_healthy` clause. 

## Debugging

1. To show standard output of the container:
```
docker logs -f <container-reference>
```
2. To run a shell in a container while container process is running:
```
docker exec -it <container-reference> /bin/bash
```
3. To start a container from an image and run a shell instead of the default entrypoint
```
docker run --rm -it --entrypoint=/bin/bash <image-reference>
```
4. To run a shell in a container after the container has stopped
```
# save container as an image
docker commit <container-ref> <new-image-name>
# then do the same as 3.
docker run --rm -it --entrypoint=/bin/bash <new-image-name>
```
5. Copying files from a (running or stopped) container to the host system
```
docker cp <container-ref>:/path/to/source/ /path/to/target
```

## Usage tips

```bash
# getting container id
CID=$(docker run -d ubuntu:18.04) # outside container
docker run -d ubuntu:18.04 --cidfile=cidfile 
cat /cidfile # outside container, blocks container startup if same file exists
cat /proc/1/cpuset | cut -c9- # inside container (impl dependent?)
hostname # inside container, impl independent but can be broken by hostname renaming
```