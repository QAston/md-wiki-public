Dockerizing Applications
================

## Instructions

## Configuring applications

To properly configure your application in docker (and for [nautilus](/infrastructure/nautilus)) you first need to be aware of the resources used by your application and you need to know how to configure the resource usage.  This section details how to get the information you need and how to pass the needed values to Nautillus.  Note that fine detail (such as where exactly to put runtime arguments) will be dependent on the application.

### Cpu

First, gather information about how many CPUs your application needs to run effectively. Once you know that you need to tell the JVM how many processors you have assigned to it. 

JVM needs this information because JDK and many libraries scale thread counts (and other cpu usage) using `Runtime.getRuntime().availableProcessors()`, which will by default be set to the number of CPUs found on the machine (hyperthreading counts as 2x, in old jvms this bypasses docker limits). Failure to configure `Runtime.getRuntime().availableProcessors()` will result in application creating too many threads which will increase the resource usage (ram, cpu) and slow the application down at the same time (more lock contention, more context switching).

Configuring `Runtime.getRuntime().availableProcessors()`:
* **Recommended** (if available): Use docker CPU limits (`--cpus` flag, docker-compose/kubernetes/OSI cpu settings, etc)
    * In Java8u131+ `availableProcessors()` are set automatically to match limits set in docker cgroups, otherwise default/manual config will be used
    * If you configure threads using this method, **make sure to put cpu configuration in your docker-compose configurations**, otherwise all applications in your docker-compose project will think they should scale threads to all your processors!
* Use manual configuration
    * In Java10+ you can configure `availableProcessors()` globally using `-XX:ActiveProcessorCount=n`
    * In older jvms you need to individually replace usages of `availableProcessors()`:
    * `-XX:CICompilerCount=n` if (`XX:CICompilerCountPerCPU=true`, which is set to false in java6 but true in java8)
    * `-XX:ParallelGCThreads=n` (if your GC uses that option, `-XX:+UseParallelGC` (default in java6), `-XX:+UseParNewGC` and `-XX:+UseG1GC` use it)
    * `-XX:ConcGCThreads=n` (if your GC uses that option, `XX:+UseG1GC` in java8 uses it)
    * `-Djava.util.concurrent.ForkJoinPool.common.parallelism=n` configures default fork-join pool size (java7+)
    * Replace remaining usages of `availableProcessors()` with either constants or configuration (for example a with reading a custom property, like `-Dcpu.scaling=n`), for example when using https://netty.io/4.1/api/index.html?io/netty/channel/nio/NioEventLoopGroup.html, use the parameterized constructors instead of defaults
* Run process with `-XX:+UnlockDiagnosticVMOptions -XX:+PrintFlagsFinal` to get the final values at startup

### Memory

To properly dockerize an application you'll need to know total memory usage of the application and other processes running in the container. 

If you don't configure the memory usage you will experience:

* java reserving too much for heap memory, slowing down the GC
* application being killed by docker on heap resizes
* performance issues caused by frequent heap resizes

Total JVM process memory usage is a sum of the following (for more technical detail see also [this post](https://plumbr.io/blog/memory-leaks/why-does-my-java-process-consume-more-memory-than-xmx)):
* [java heap](#java-heap-1), configured using `-Xms1024M`(startup size) and `-Xmx1024M` (max size)
* total number of threads * stack-size, stack-size configured using `-Xss1024K` (the defaults vary between OSes)
* metaspace (java8+) - used to store class objects, `-XX:MetaspaceSize=` `-XX:MaxMetaspaceSize=` configure the initial and max size (max size is unlimited by default). Metaspace by default has a subsection for [compressed oops](http://java-latte.blogspot.com/2014/03/metaspace-in-java-8.html) which by default has max size of 1G and can be configured using `-XX:CompressedClassSpaceSize=`. [Metaspice resize triggers a full gc](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/considerations.html), so making initial size bigger (or equal to max) might potentially speed up startup time.
* permgen (pre-java8) - used to store class objects and interned strings, configured using `-XX:PermSize=N` (initial) and `-XX:MaxPermSize=N` (max size), default max size is OS dependent but around 64MB-100MB. Permgen resize triggers a full gc, so making initial size bigger (or equal to max) might potentially speed up startup time.
* "direct memory" - native memory used by java.nio (and some libraries like netty), configured using `-XX:MaxDirectMemorySize=`, by default has size equal to size of `Rumtine.getRuntime().maxMemory()` (which roughly returns size of java heap), see also [this post](https://dzone.com/articles/default-hotspot-maximum-direct-memory-size)
* native memory allocated by `java` process, this includes `java` internals and JNI allocations (includes libraries like Netty). There's no switch to configure those, you can inspect the values using `-XX:NativeMemoryTracking=detail` (jdk8+).

From the above list **you have to explicitly configure the [java-heap](#dockerizing-applications_configuring-applications_memory_java-heap)** (otherwise GC will not work properly). Other values don't have to be explicitly configured because they don't use a GC mechanism, but you still have to be aware how much memory they take. The easiest method to obtain the values is running the application and getting the RSS memory (for example using `top`) to get total space and [JMX](/tools/jmx) to get the size of each memory pool.

#### Java Heap

Java heap is where jvm stores objects allocated with `new` and it the space that's managed by GC. You can get the heap size at runtime using `Rumtine.getRuntime().maxMemory()` (be mindful of some [caveats](https://plumbr.io/blog/memory-leaks/less-memory-than-xmx)).

Be careful about choosing your heap size. The bigger the heap-size the longer the full GC cycles will be, so more doesn't always mean better. Also, different GCs are better for different heap sizes, see a basic introduction [here](https://www.baeldung.com/jvm-garbage-collectors).

Configuring heap size:
* **Recommended:** Set heap sizes manually:
    * Max heap is set using `-Xmx` for example: `-Xmx1024M`
    * Startup heap is set using `-Xms`, for example `-Xms1024M`
    * Heap resizing is a relatively slow operation, so if you want fast startup it might be a good idea to configure the start size to be big, even equal to max size.
* **Not recommended** Use `MaxRAM`:
    * By default `-XX:MaxRAM=` is the total amount of ram found on the machine and MaxHeapSize is set to `-XX:MaxRam=`/`-XX:MaxRAMFraction=`(which defaults to 4). Both of those values can be overridden.
    * In java8131 when running with `-XX:+UnlockExperimentalVMOptions` and `-XX:+UseCGroupMemoryLimitForHeap` and in java10 by default `-XX:MaxRAM=` will be set to the docker memory limit (this is equivalent to setting -XX:MaxRAM=`cat /sys/fs/cgroup/memory/memory.limit_in_bytes` or nothing if docker limits are not configured). The value can't change after startup, so the limit set on startup will be used.
    * The only purpose of `-XX:MaxRAM=` is to be used in the default heap size calculation. Overriding the value will NOT affect the total jvm process usage, thread memory or anything other memory.
    * Scaling with system ram is a bad idea for apps running in different environments
    * Some sources advise using `-XX:MaxRAMFraction=1` and `-XX:+UseCGroupMemoryLimitForHeap` for docker (this includes SRE docker plugin), DON'T DO THIS as this assumes your application only uses heap memory (which is never true) and will cause your application to be killed by docker.

* Run process with `-XX:+UnlockDiagnosticVMOptions -XX:+PrintFlagsFinal` to get the final values at startup

### More on java/docker resource tuning

* [process memory usage breakdown - resident vs committed memory](https://stackoverflow.com/questions/31173374/why-does-a-jvm-report-more-committed-memory-than-the-linux-process-resident-set#answer-31178912)
* [how cpu/(resident)memory constraints are applied to containers](https://docs.docker.com/config/containers/resource_constraints/) 
* [docker’s usage of cgroups](https://docs.docker.com/config/containers/runmetrics/#control-groups)
* [java's support for docker in java8u131](https://blogs.oracle.com/java-platform-group/java-se-support-for-docker-cpu-and-memory-limits)
* [java gc implementation overview](https://www.baeldung.com/jvm-garbage-collectors)
* [g1gc tuning](https://docs.oracle.com/javase/9/gctuning/garbage-first-garbage-collector-tuning.htm#JSGCT-GUID-379B3888-FE24-4C3F-9E38-26434EB04F89)
* [basic java flags](https://docs.oracle.com/javase/8/docs/technotes/tools/windows/java.html) and [extended java flags](http://jvm-options.tech.xebia.fr/) and [permgen/metaspace flags](https://dzone.com/articles/permgen-and-metaspace)

### Logging

The logging method used depends on whether [ELK](https://sre.pages.tech.lastmile.com/services/elk.html) is available in the environment you want to run your application. If you want your application to work in all environments you need to provide multiple config for both ELK and filesystem logging and use right config for each [environment configuration](#dockerizing-applications_configuring-applications_environment-configuration). 

Switching between config files is usually done using a system property:
* log4j1 - `-Dlog4j.configuration=` config file in filesystem (or classpath if not found in filesystem).
* log4j2 - `-Dlog4j.configurationFile=` config file in filesystem (or classpath if not found in filesystem).
* spring boot - `-Dlogging.config=classpath:log4j-logstash.xml`

#### Logging: Nautilus ELK (service inventory)

It is bad practice to write the logs to the filesystem when on Nautilus. This is because the Nautilus filesystem is difficult (or impossible) to access.  In addition getting your docker container would have to make sure you don't overflow the assigned volume space. This would have to be done by gzipping and deleting obsolete logs, which requires a lot of individual configuration for each container and consumes a lot of disk space.

For service-inventory deployments (CIT/PROD) write the logs to stdout in the logstash format.

To use logstash pick one of the libraries given below and follow the linked instructions.  Which one you should use will depend on the specifics of the application you are dockerizing, most notably the version of Java.

* **log4j1**: Follow [https://github.com/logstash/log4j-jsonevent-layout](https://github.com/logstash/log4j-jsonevent-layout).
* **log4j2.10+**: Follow [https://github.com/vy/log4j2-logstash-layout](https://github.com/vy/log4j2-logstash-layout).
    * This is Java 8+ only.
    * Use -fatjar artifact to avoid dependency conflicts. 
    * Make sure to set prettyPrintEnabled="false" config in log4j configuration, otherwise logs will be multiline and logstash will not parse them
* **log4j2.3**: Follow [https://gitlab.tech.lastmile.com/classic-control-systems/log4j2-logstash-layout-java6](https://gitlab.tech.lastmile.com/classic-control-systems/log4j2-logstash-layout-java6).
    * This is the latest version of log4j2 that supports java6. It works with java8 too.
    * Use -fatjar artifact to avoid dependency conflicts. 
* **logback**: Follow [https://sre.pages.tech.lastmile.com/services/elk.html#logback](https://sre.pages.tech.lastmile.com/services/elk.html#logback).
    * Logback will pick up things logged to stdout as plaintext, which is useful for logging from a bash script.
* **No Logger**: It is possible to not use a logger at all and print directly to stdout.
    * Logs output in this way will not have all the metadata and don't have multiline support (which is needed for things like stacktraces).
* **Multiple Loggers**: There is nothing code wise to prevent you from using multiple loggers in the same application, or from using loggers alongside printing to stdout.

Once this is done logs can then be accessed using [ELK web interface (see Instances)](https://sre.pages.tech.lastmile.com/services/elk.html) or [elasticsearch-log-viewer](https://gitlab.tech.lastmile.com/internal-open-source/elasticsearch-log-viewer).

Note: [ELK has limits](https://sre.pages.tech.lastmile.com/services/elk.html#limits); if you log too much the logs will be queued and thread which logs to stdout will be slowed down.

#### Logging: Filesystem

If there's no ELK service available in the environment and writing logs to filesystem isn't a problem, write the logs to `/logs` directory (the easiest way to do this is to create a symplink to the directory using `RUN ln -s /logs <application-logdir>`, ). The directory was chosen so that it's standardized between applications and easy to mount to the host filesystem as a bindmount.

### Filesystem

Docker uses a [CopyOnWrite filesystem](https://docs.docker.com/storage/storagedriver/) by default, which has the following effects:
* All i/o operations go through a layer of indirection (COW driver), which incurs a (small) performance penalty on every operation
* The first modification to an existing file is slow because the file has to be copied (the larger the file the longer it takes)
* Changes to the files remain there as long as the exact same container is being restarted, but are lost after container is removed or recreated (no persistence)
* The more layers the more inefficient the filesystem is.

Docker provides a [volume/bindmount mechanism](https://docs.docker.com/storage) for applications which need persistence or fast i/o. Make sure to document in container README which directories need to be mounted as a volume/binmount for persistence and optimal performance. 

To configure the volumes you can use:
 * `-v` flag [in docker](https://docs.docker.com/storage/volumes/#choose-the--v-or---mount-flag)
 * `volumes:` [declarationin in docker-compose](https://docs.docker.com/compose/compose-file/#volumes)
 * `persistent_volumes_size` [declaration in service-inventory](https://sre.pages.tech.lastmile.com/platform/storage.html). 


### Network

#### External services

All services from outside Ocado (for example: aws, google big data) that your application connects to count as external services. PROD/CIT servers don't allow connecting to external services except through `proxy.ocado.com` which is an http proxy.

To globally configure http/https proxy for a jvm you can use the following system properties: `-Dhttp.proxyHost=proxy.ocado.com -Dhttp.proxyPort=8080 -Dhttps.proxyHost=proxy.ocado.com -Dhttps.proxyPort=8080`. Many java http client libraries also allow configuring the proxy http request instead of globally if needed.

#### Exposed services

All incoming connections accepted by your application count as exposed services. 

There are several patterns that services follow:
* **Recommended**: If possible your service should simply listen on a static port (which is the case most of the time). If that's the case all you need to do is [adding an EXPOSE directive to the dockerfile](#dockerizing-applications_writing-dockerfiles_container-processes_entrypoint-declaration) and maybe document what service the port is providing.
* If your service listens to random ports, change the service to listen to a static port instead as there's no way to make exposing random ports work in as ports are being exposed before the application is started.


#### Network references

To allow your application to be deployed in any network configuration (docker-compose, service-inventory, mixed environments) you need to make sure that all variable network references used (URLs, IPs, DNSes, ports, etc) are configurable using env variables. **Failure to make network references configurable will make your docker image unusable outside of it's hardcoded network layout**.
The configuration should be done through [environment variables](#dockerizing-applications_configuring-applications_environment-configuration) and follow the naming conventions:
* URLs should be configurable using `<NAME>_URL` env variable convention, example: `ACTIVEMQ_URL`, `DB_URL`
* Hostnames(DNS names/IPs) and ports should be configurable using `<NAME>_HOST` and `<NAME>_PORT` env variable convention, example: `THUG_HOST`, `THUG_PORT`.
* References (URLs/Hostnames/ports) which are sent to clients outside of the docker network should be configurable using env variables with `PUBLIC_` prefix. This reference can then be configured to the NAT/ingress adress or the mapped port and allow sending the clients links that work from their network.
    * Example: 2 reporting services are in a single compose network. If one service wants to query another it uses `<SERVICENAME>_URL`. If the service wants to send a link to another service to the client browser, it sends the `PUBLIC_<SERVICENAME>_URL` link, as browsers are never in the docker network. Alternatively a setup with a reverse-proxy can be used (see http/https reverse proxy headers: https://www.nginx.com/resources/wiki/start/topics/examples/forwarded/ )
    * This solution has a limitation - a single NAT/ingress/port-mapping has to work for all clients. To work around that you need to know which client is coming from which network and send appropriate reference accordingly.
* Back-references to the container itself are a special case:
    * Back-references used privately by the container itself should be done as usual using `localhost` and [exposed port](#dockerizing-applications_writing-dockerfiles_container-processes_entrypoint-declaration) numbers, no need for configuration
    * Back-references sent to other containers within the same docker network should use container hostname (in java `InetAddress.getLocalHost().getHostName()`). This value is configured automatically by most docker environments (`--hostname`).
    * Back-references sent to **HTTP/HTTPS** clients should use a reverse proxy which sets forwarding headers, the application needs to be configured to handle the headers, see: https://www.nginx.com/resources/wiki/start/topics/examples/forwarded/ and [nautilus ingress configuration (search for "configuring ingress")](/infrastructure/nautilus#nautilus_system-definitions_adding-a-system)
    * Back-references sent to other clients outside the docker network should be configurable using `PUBLIC_HOSTNAME` (also `PUBLIC_<NAME>_PORT` or `PUBLIC_<NAME>_URL` if needed).


### Environment Configuration

Most of the applications need to be able to run in multiple different envirionments (Prod, CIT, docker-compose, VIP, Nautilus, etc). The preferred approach to making applications run in different environments is a combination of 2 approaches:
* Packaging configuration files with the application and selecting a config profile/mode using `$MODE` environment variable (docker) or launch files (VIPs)
* Overriding individual settings using environment variables (optional, `$MODE` should provide the defaults)

As a fallback if the above approach isn't sufficient you can override the config files in [Nautilus/service-inventory definition](https://sre.pages.tech.lastmile.com/platform/service-inventory.html) or if even that doesn't work you can always build environment-specific docker images (as last resort).

#### Configuration using $MODE and env variables

* The docker container should use `MODE` environment variable to switch the environment profile/mode used, if `MODE` is not specified the default `dev` configuration should be used
* `MODE` values should follow this convention: `<env-type(dev/cit/prod/etc)>_<instance(optional disambigulation)>_<cfc(optional disambigulation)>` (all lower case) , examples: `dev`, `cit_cfc2`, `staging_cfc1`, `prod`, `prod_ambzone_cfc1`
    * `dev` profiles are the profiles for use when running locally using docker and docker-compose, it should be convenient for a typical docker setup to reduce the number of environment variables that needs to be passed in docker-compose files
    * `cit`/`prod`/`staging` - self explanatory; for use from launch files or docker, depending on which platform is the instance
* Configuration for modes which run within dockerized environments (especially `dev`) should use environment variables to provide config overriding mechanism for settings, specifically:
    *  [network references](#dockerizing-applications_configuring-applications_network_network-references)
    *  Java arguments and system properties:
        *  `JAVA_OPTS` - jvm performance options
        *  `APP_OPTS` - application-specific flags
        *  `LOGGING_OPTS`, `JMX_OPTS`, `NEWRELIC_OPTS`, etc - java configuration options grouped per category

Implementing `MODE`s and environment variable configuration:
* [spring boot](#dockerizing-applications_configuring-applications_environment-configuration_configuration-using-mode-and-env-variables_mode-and-env-variables-in-spring-boot)
* [CommonUtilities/config-loader-utilities](#dockerizing-applications_configuring-applications_environment-configuration_configuration-using-mode-and-env-variables_mode-and-env-variables-in-commonutilities-25-2-0-generalconfig-and-in-config-loader-utilities)
* [apollo](#dockerizing-applications_configuring-applications_environment-configuration_configuration-using-mode-and-env-variables_env-variables-in-apollo-and-other-properties-configurations)
* Configuration not covered by the above can be implemented in [the entrypoint script](#dockerizing-applications_writing-dockerfiles_container-processes_entrypoint-script) or directly in the application (for example by reading env variables using `System.getenv()`.

##### $MODE and env variables in spring boot

Springboot's `application.yml` supports the following profile syntax (see [Documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-properties-and-configuration.html)):

```yml
---
spring:
    profiles: prod_cfc1

server:
    address: pewar311

broker:
     - failover:(${ACTIVEMQ_URL?tcp://activemq:61616})
---
spring:
    profiles: prod_cfc2
# rest of the config here
```
 
Refering to env-variables/properties (there's no distinction) is done using `${name:default-value}` syntax, where `name` is a name of a system property or environment variable. The property lookup is implemented using [StandardEnvironment](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/env/StandardEnvironment.html) class by default, it can be changed to use a different class).

`MODE`s are declared using `spring.profiles` key, to select the mode set `-Dspring.profiles.active=${MODE}` java argument.


##### $MODE and env variables in CommonUtilities 25.2.0+ GeneralConfig and in config-loader-utilities

Version 25.0.1 (or newer) CommonUtilities supports both modes and environment variables when using yaml config files. Follow [this guide](https://gitlab.tech.lastmile.com/classic-control-systems/CommonUtilities#yaml-global-config) for instruction on configuration and updates.

[config-loader-utilities](https://gitlab.tech.lastmile.com/classic-control-systems/config-loader-utilities.git) supports the same config format and configuration but doesn't force you to pull entire CommonUtilities library for it.

You can switch modes using `-Dmode=${MODE}`. You can use `${env:ENV_VAR_NAME?default-if-empty}` notation in the config to use environment variables (and `${sysprop:ENV_VAR_NAME?default-if-empty}` to use system properties).

You can find an example config.yml [here](https://gitlab.tech.lastmile.com/ocean-cfcs-outbound/ReverseFlows/blob/master/src/main/resources/config.yml) and [here](https://gitlab.tech.lastmile.com/inbound-cfc/ApplicationMonitor/blob/master/src/main/resources/config.yml).


##### Env variables in apollo (and other .properties configurations)

You can use `${ENV_VAR_NAME}` notation to use environment variables in `apollo.properties`, [see this example](https://gitlab.tech.lastmile.com/classic-control-systems/mis-admin-cfc1/blob/master/src/main/resources/db.wms.properties)

For that to work you need to use [`EnchancedProperties` class](https://gitlab.tech.lastmile.com/cfc-apollo/apollo-common/blob/master/src/main/java/com/ocado/apollo/util/EnhancedProperties.java) instead of standard properties class to load the config files.

Enchanced properties do not provide a mechanism for defining multiple modes in a single file, but you can pass a different file using a system property in [the entrypoint script](#dockerizing-applications_writing-dockerfiles_container-processes_entrypoint-script).


## Writing dockerfiles

Next step after knowing how your application works is creating a Dockerfile which is executed by `docker build` to create the image.

Typical Dockerfiles follow this structure:

1. [Base image declaration](#dockerizing-applications_writing-dockerfiles_base-image)
2. [Populating the image's filesystem](#dockerizing-applications_writing-dockerfiles_filesystem)
3. [Configuration of the main process](#dockerizing-applications_writing-dockerfiles_container-processes)
4. [Metadata declaration](#dockerizing-applications_writing-dockerfiles_metadata)

An example Dockerfile looks like this:
```
FROM mirror-releases.docker.ocean.tech.lastmile.com/ocean_cfc/sun-jre-1.6.0_26-debian-lenny:1.2.1

ADD target/ApplicationMonitor.tar.gz /app
ADD docker/docker-entrypoint.sh /app

RUN ln -s /logs /app/logs && chown -R 65534:65534 /app

# Ports: JMXMP: 9002
EXPOSE 9002
USER 65534
WORKDIR /app
ENTRYPOINT ["./docker-entrypoint.sh"]

# Put this at the end to let the statements above be cached (as BUILD_DATE changes with every build)
ARG GIT_URL=unspecified
ARG GIT_COMMIT=unspecified
ARG BUILD_DATE=unspecified
ARG APP_VERSION=unspecified
LABEL org.label-schema.schema-version="1.0" \
  org.label-schema.name="ApplicationMonitor"\
  org.label-schema.description="ApplicationMonitor"\
  org.label-schema.vendor="Ocean CFC Systems"\
  org.label-schema.maintainer="classic-control-systems-developers-xd@ocado.com" \
  org.label-schema.build-date="$BUILD_DATE" \
  org.label-schema.vcs-ref="$GIT_COMMIT" \
  org.label-schema.vcs-url="$GIT_URL" \
  org.label-schema.version="$APP_VERSION"
```

### Base image

Choosing the base image:
* MUST be an image from a repository which doesn't allow overwriting artifacts to make sure the builds are deterministic:
    * releases: `releases.docker.ocean.tech.lastmile.com/ocean_cfc/` 
    * mirror-releases: `mirror-releases.docker.ocean.tech.lastmile.com/ocean_cfc/`
    * mirror-hub: `mirror-hub.docker.tech.lastmile.com` - proxy for https://hub.docker.com/
    * [other mirrors](https://gitlab.tech.lastmile.com/devops/nexus-infra/blob/master/README.md#proxy-registries)
* Use the following base images for testing applications deployed to prod VIPs (intended to match prod setups as much as possible):
    * Java6: https://gitlab.tech.lastmile.com/ocean-cfc-systems/thirdparty-images/sun-jre-1.6.0_26-debian-lenny
    * Java6 Tomcat5: https://gitlab.tech.lastmile.com/ocean-cfc-systems/thirdparty-images/tomcat-5-jre-6
    * Java6 Glassfish3: https://gitlab.tech.lastmile.com/ocean-cfc-systems/thirdparty-images/glassfish3
    * Java8: https://gitlab.tech.lastmile.com/ocean-cfc-systems/thirdparty-images/jre-8-ubuntu-14.04
    * Java8 Tomcat7: https://gitlab.tech.lastmile.com/ocean-cfc-systems/thirdparty-images/tomcat-7-jre-8
    * Java11: https://gitlab.tech.lastmile.com/ocean-cfc-systems/thirdparty-images/jdk-11-alpine
* Otherwise you can use [example images suggested by SREs](https://sre.pages.tech.lastmile.com/platform/topic/base-images.html) or other images as long as they don't violate ocado and ocean cfc guidelines.
* Do not use base images from [CommonDockerFiles](https://gitlab.tech.lastmile.com/classic-control-systems/CommonDockerFiles) because they're not versioned and would cause nondeterministic builds

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
    * `--expose=<containerport>` - expose port to the docker network in which container is running
    * `--hostname` - override hostname of the container
    * `--pid=` - override PID of the process(`1` by default)
    * `--user=<uid>[:<gid>]` - override USER running the process
    * `--init=true` - create an init process for managing children processes and forwarding signals to them (potentially makes writing entrypoints easier, but don't rely on this as it's not availabe in Nautilus) 
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

#### Entrypoint script

By docker convention the script is usually named `docker-entrypoint.sh`.

Tasks performed by the entrypoint script (the [main process of the container](#dockerizing-applications_writing-dockerfiles_container-processes_main-container-process)):

1. Setting configuration and defaults which aren't configured elsewhere:
    * setting defaults for each $MODE
        * make sure the defaults are overridable by only setting them if the variable is undefined, this can be done using `${VARNAME:-"PUT DEFAULTS HERE"}` syntax
        * defaults should be grouped in env variables contained related settings to enable overriding each value separately
        * mode-specific defaults can be handled using a `case` statement
    * setting java flags, for example:
        * classpath, config file paths, system properties, etc.
        * `-Duser.name=` and `-Duser.home=` if your application relies on `user.name` or `user.home` system properties (otherwise `nobody` user name and an OS-dependent path will be used which probably isn't what you want)
        * [cpu](#dockerizing-applications_configuring-applications_cpu) and [memory](#dockerizing-applications_configuring-applications_memory) flags
        * other performance tweaks like `-Djava.security.egd=file:/dev/./urandom` [(why is "." needed)](http://www.thezonemanager.com/2015/07/whats-so-special-about-devurandom.html)
        * [jmx configuration for monitoring/profiling](/tools/JMX#jmx_connecting-to-a-docker-image-or-through-any-nat-port-forwarder)
        * if in `prod` or `cit`: [newrelic agent](/tools/NewRelic/development-config)
        * if application is running in cit and uses opennnms: [opennms udp forwarder](https://gitlab.tech.lastmile.com/classic-control-systems/opennms-forwarder) (needed because Nautilus doesn't support exposing udp ports)
        * set proxy flags if you need to connect to [external services](/tools/docker/dockerizing-applications#dockerizing-applications_configuring-applications_network_external-services)
        * other flags, testing environments should aim to match production flags if possible
2. Starting the application process(es), waiting until the processes are finished, and forwarding signals to the processes to signal termination 
    * The recommended method is using `exec <process>` as the last command in the script, see [exec declaration](http://wiki.bash-hackers.org/commands/builtin/exec)
    * If `exec <process>` can't be used you need to start the process in the background, trap SIGINT and SITGERM, and trigger process cleanup upon receiving those signals, see [forwarding signals in bash](https://unix.stackexchange.com/questions/146756/forward-sigterm-to-child-in-bash)
    * If you fail to use the above methods your process will not be asked to shut down before container termination because bash (and other shells) doesn't forward signals to subprocesses by default

Example entrypoint script:
```bash
#!/bin/bash

# a workaround that enables mounting host directories using --user=0 --volume=`pwd`/logs:logs from non-root containers
source /util/ensure-bindmount-user.sh /logs

# handle config modes/profiles
MODE=${MODE:-dev_cfc1}
case $MODE in
cit*)
    # opennms forwarder
    export OPENNMS_HOST=${OPENNMS_HOST:-"127.0.0.1"}
    export OPENNMS_PORT=${OPENNMS_PORT:-"10062"}
    export OPENNMS_FWDTARGET_HOST=${OPENNMS_FWDTARGET_HOST:-"ocsopennms.ocs-monitoring"}
    export OPENNMS_FWDTARGET_PORT=${OPENNMS_FWDTARGET_PORT:-"10062"}
    OPENNMS_OPTS=${OPENNMS_OPTS:-"-javaagent:/util/opennms-forwarder.jar"}

    # elk
    LOGGING_OPTS=${LOGGING_OPTS:-"-Dlog4j.configuration=log4j-logstash.xml"}
  ;;
esac

# set default env variable values
LOGGING_OPTS=${LOGGING_OPTS:-""}
NEWRELIC_OPTS=${NEWRELIC_OPTS:-""}
JMX_OPTS=${JMX_OPTS:-"-javaagent:/util/jmxmp-java-agent.jar"}
OPENNMS_OPTS=${OPENNMS_OPTS:-""}
JAVA_OPTS=${JVM_OPTIONS:="-Xms64M -Xmx256M -XX:ParallelGCThreads=2 -XX:CICompilerCount=2 -Djava.security.egd=file:/dev/./urandom"}
APP_OPTS=${APP_OPTS:-"-Dmode=${MODE} -Dorg.apache.activemq.SERIALIZABLE_PACKAGES=* -Djava.net.preferIPv4Stack=true -Dconfig.file=config/config.yml"}
APP_ARGS=${APP_ARGS:-"com.ocado.alerting.Register"}

exec java ${JMX_OPTS} ${OPENNMS_OPTS} ${NEWRELIC_OPTS} ${LOGGING_OPTS} ${JAVA_OPTS} ${APP_OPTS} -cp "ApplicationMonitor/*:." ${APP_ARGS}
```

#### Configuring healthcheck

You can optionally declare a healthcheck helper process to be periodically run to publish the status of the main process:
* this declaration is optional, as standalone docker uses the main process to verify that container is running. Status of HEALTHCHECK is displayed in docker ps, but ignored by docker process.
* Nautilus (service-inventory) [has it's own healthcheck declaration](https://sre.pages.tech.lastmile.com/platform/checks.html) which probably overrides the ones defined in Dockerfiles.
* [docker-compose v2.1](https://docs.docker.com/compose/compose-file/compose-file-v2/#depends_on) will delay startup of dependent containers until dependencies are started, if requested using `depends_on.condition: service_healthy` clause. 


### Metadata

Metadata is the last declaration in the image, in ocean cfc images it should follow this convention:
```
ARG GIT_URL=unspecified
ARG GIT_COMMIT=unspecified
ARG BUILD_DATE=unspecified
ARG APP_VERSION=unspecified
LABEL org.label-schema.schema-version="1.0" \
  org.label-schema.name="ApplicationMonitor"\
  org.label-schema.description="ApplicationMonitor"\
  org.label-schema.vendor="Ocean CFC Systems"\
  org.label-schema.maintainer="classic-control-systems-developers-xd@ocado.com" \
  org.label-schema.build-date="$BUILD_DATE" \
  org.label-schema.vcs-ref="$GIT_COMMIT" \
  org.label-schema.vcs-url="$GIT_URL" \
  org.label-schema.version="$APP_VERSION"
```

The declaration is at the end of the file to make layer caching more effective (BUILD_DATE changes every build).


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
6. Attaching java debugger
   * Add `-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5050` to entrypoint java flags
   * Expose the port to host, figure out the address of the container visible from your machine.
   * Connect to the exposed port using java debugger in your IDE

## Continuous Integration

Your application should use [ci-core](https://gitlab.tech.lastmile.com/classic-control-systems/ci-core/tree/master/templates) for continuous integration, if your application is currently on jenkins you need to migrate to gitlab.

Gitlab will generally build the following images:
* releases to `releases.docker.ocean.tech.lastmile.com/ocean_cfc/${APP_ID}:${APP_VERSION}`
* branches to `internal.docker.ocean.tech.lastmile.com/ocean_cfc/${APP_ID}:ci-${BRANCH}-latest`

The exact directory layout and expected scripts depend on the exact [ci-core](https://gitlab.tech.lastmile.com/classic-control-systems/ci-core/tree/master/templates) template used. This guide focuses on the typical case where [jdkX-maven/gradle-application templates](#dockerizing-applications_continuous-integration_application-templates) are used.

### Application templates

Projects using `jdkX-maven/gradle-application` templates have to use the following directory structure (otherwise gitlab will not find the scripts):
* /docker/
    * [dockerBuild.sh](#dockerizing-applications_continuous-integration_application-templates_docker-dockerbuild-sh) - builds an image (you need to compile/package it yourself first)
    * [dockerPush.sh](#dockerizing-applications_continuous-integration_application-templates_docker-dockerpush-sh)
    * [docker-entrypoint.sh](#dockerizing-applications_writing-dockerfiles_container-processes_entrypoint-script)
    * [Dockerfile](#dockerizing-applications_writing-dockerfiles)
    * [docker-compose.yml](#dockerizing-applications_continuous-integration_application-templates_docker-docker-compose-yml) (optional) - a [docker-compose](https://docs.docker.com/compose/) configuration for local testing
    * dockerCompose.groovy (optional) - a convenience script for running `docker-compose` based on [DockerScriptsLibrary](https://gitlab.tech.lastmile.com/classic-control-systems/DockerScriptsLibrary) version 2+

Remember to make all the `*.sh` files executable using `git update-index --chmod=+x docker/*.sh`, otherwise the files will not be executable on unix machines cloning the repository.

You can use a [gradle-plugin](https://gitlab.tech.lastmile.com/SRE/devtools/sre-platform/blob/master/sre-platform-gradle/src/main/groovy/com/ocado/sre/platform/DockerPlugin.groovy) instead of dockerBuild.sh/dockerPush.sh, but when doing so make sure you provide your own Dockerfile, as the defaults in the plugin are incorrect (see [java heap](#dockerizing-applications_configuring-applications_memory_java-heap) section). In the future a maven plugin will also be available.

#### docker/dockerBuild.sh

This script serves 2 purposes:
* developers can run `docker/dockerBuild.sh` to quickly build development docker images with the following tag: `internal.docker.ocean.tech.lastmile.com/ocean_cfc/<image-name>:dev-${BRANCH:-$USER}`, for example: `internal.docker.ocean.tech.lastmile.com/ocean_cfc/application_reporting:dev-ccsob1555`. `${USER}` will be used instead of `${BRANCH}` for master branch and when there's no current branch available in the repository.
* gitlab pipelines will use the script to build images with tag and version specified by ci-core:
    * development branches: `internal.docker.ocean.tech.lastmile.com/ocean_cfc/${APP_ID}:ci-${BRANCH}`
    * releases: `releases.docker.ocean.tech.lastmile.com/ocean_cfc/${APP_ID}:${RELEASED-VERSION}`
    * by default `${APP_ID}` is the name of the gitlab repository, you can override this using a ci-core variable in [config.yaml](https://gitlab.tech.lastmile.com/classic-control-systems/ci-core/blob/master/config.yaml)

The script should have the exact following contents (with `DEFAULT_TAG` set apporopriately):
```bash
#!/bin/bash

#
# Script to build docker images with the given image tag given as a single argument.
# When no arguments are given builds with ${DEFAULT_TAG}:dev-${BRANCH} version
# When not on a branch, or when branch is "master", the $USER will be used instead to avoid collisions and confusion
# The script assumes the application is already packaged (for example using `mvn clean package`
#

DEFAULT_TAG=internal.docker.ocean.tech.lastmile.com/ocean_cfc/application_monitor

BRANCH=`git rev-parse --abbrev-ref HEAD | tr -d [:space:]`
if [ "$BRANCH" == "master" ] ||  [ "$BRANCH" == "HEAD" ]; then
  BRANCH=$USER
fi

TAG=${1:-"$DEFAULT_TAG:dev-$BRANCH"}

echo "Building image with tag: $TAG"

THIS_SCRIPT_DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"
if [[ `uname -s` =~ CYGWIN.* ]]; then
  THIS_SCRIPT_DIR=`cygpath -w ${THIS_SCRIPT_DIR}`
fi

cd ${THIS_SCRIPT_DIR}
PROJECT_DIR="$THIS_SCRIPT_DIR/.."

DATE_CMD=${DATE_CMD:-date} # This gives MacOS users opportunity to set DATE_CMD to 'gdate'

check_for_error() {
  if [[ $? -ne 0 ]]; then
    echo "ERROR: $1"
    exit 1
  fi
}

BUILD_DATE=$(${DATE_CMD} -Iseconds)
docker build --build-arg GIT_URL="${GIT_URL:-`git ls-remote --get-url origin`}" \
      --build-arg GIT_COMMIT="${GIT_COMMIT:-`git rev-parse HEAD`}" \
      --build-arg BUILD_DATE="${BUILD_DATE}" \
      --build-arg APP_VERSION="${APP_VERSION:-unspecified}" \
       -t $TAG \
       -f $THIS_SCRIPT_DIR/Dockerfile \
       ${PROJECT_DIR}
check_for_error "Failed to build the image"
```

#### docker/dockerPush.sh

This script pushes the image build with `dockerBuild.sh` given the same arguments.

The script should have the exact following contents (with `DEFAULT_TAG` set apporopriately):
```bash
#!/bin/bash
#
# Script to push docker images with the given image tag given as a single argument.
# When no arguments are given builds with ${DEFAULT_TAG}:dev-${BRANCH} version
# When not on a branch, or when branch is "master", the $USER will be used instead to avoid collisions and confusion
# The script assumes the image is already built using dockerBuild.sh
#

DEFAULT_TAG=internal.docker.ocean.tech.lastmile.com/ocean_cfc/application_reporting

BRANCH=`git rev-parse --abbrev-ref HEAD | tr -d [:space:]`
if [ "$BRANCH" == "master" ] ||  [ "$BRANCH" == "HEAD" ]; then
  BRANCH=$USER
fi

TAG=${1:-"$DEFAULT_TAG:dev-$BRANCH"}

echo "Pushing image with tag: $TAG"

docker push $TAG
```

#### docker/docker-compose.yml

An optional file with local development configuration using [docker-compose](https://docs.docker.com/compose/):
* Use docker-compose format 2.1 or newer
* Applications using a db or activemq and [docker-compose between 2.1 and 3.0](https://docs.docker.com/compose/compose-file/compose-file-v2/#depends_on) should declare a startup dependency using `depends_on` clause (links clause is obsolete and will not work correctly in this case) - docker-compose will wait for these services to pass their healthcheck before starting the containers that depend on them:
```yml
depends_on:
      db:
        condition: service_healthy
      activemq:
        condition: service_healthy
```
* If you're using docker-compose 3.0+, application dependencies should be solved by first running `docker-compose up -d db activemq` and then running `docker-compose up -d` to start the rest of the services.


## Migrating applications dockerized using old guidelines

If your project was dockerized according to the [old guidelines](https://gitlab.tech.lastmile.com/classic-control-systems/ocean-cfc-documentation/blob/7090a5d4e5ff178802ba9082efa748ebaf1d5570/tools/dockerizing-applications.md) need to be reviewed and redockerized.

In particular:

1. The application needs to be migrated to [new nexus](/tools/Nexus#migration) and [ci-core](https://gitlab.tech.lastmile.com/classic-control-systems/ci-core):
    * `jenkins/` directory and jenkins jobs need to be deleted
    * [CI config need to be redone](#dockerizing-applications_continuous-integration_application-templates) (specifically you need to use new better versions of `dockerBuild.sh`/`dockerPush.sh`)
3. [Application configuration](#dockerizing-applications_configuring-applications) needs to be reviewed and updated to make sure the application is production ready (old guidelines were dev-only), specifically:
    * [environment configuration](#dockerizing-applications_configuring-applications_environment-configuration) needs to supoort profiles/modes, overriding using env variables and necessary options need to be made configurable
    * [logging](#dockerizing-applications_configuring-applications_logging) needs to be configured to use standard `/logs` directory while in docker and needs to have ELK-specific config file variant for Nautilus
    * performance and filesystem configuration needs to be reviewed because it's most likely not configured at all
    * network configuration is likely to be incomplete
4. The Dockerfile should be [redone](#dockerizing-applications_writing-dockerfiles), specifically:
    * New, better, versioned [base images](#dockerizing-applications_writing-dockerfiles_base-image) should be used, old images are not versioned and use some obsolete workarounds
    * `dockerWrapper.sh` entrypoint script should be replaced with new [docker-entrypoint.sh](#dockerizing-applications_writing-dockerfiles_container-processes_entrypoint-script):
        * the old workarounds for configuration (editing config files) should be removed and [environment configuration guide](#dockerizing-applications_configuring-applications_environment-configuration) should be followed instead
        * the old workaround for waiting for dev activemq/db using `wait-for-it.sh` should be removed - this should be replaced by [docker-compose `depends_on: condition: service_healthy` configuration](#dockerizing-applications_continuous-integration_application-templates_docker-docker-compose-yml) in the dev environment. If you really-really want to wait for the db in prod then this should be implemented in the application recovery code or as a java agent which blocks application startup until the db (or other dependency) is available.
5. Remove unversioned docker images from new nexus (to prevent people from using obsolete artifacts):
    * log in to https://nexus.ocean.tech.lastmile.com/#browse/browse:internal and go to internal/ocean_cfc
    * delete manifests which have version `:latest` `:dailytest` `:latest-release`

### docker-compose-common.yml

Some of the projects contain a `docker-compose-common.yml` file which was used to bundle shared docker-compose definitions for [DockerScriptsLibrary groovy scripts](https://gitlab.tech.lastmile.com/classic-control-systems/DockerScriptsLibrary). This approach is now deprecated and the `docker-compose-common.yml` file should be deleted. 

This is a breaking change, so [all scripts which refer to your image](https://euw1-ocssourcegraph.eu-west-1.aws.shd.dev.lastmile.com/search?q=@Grab+com.ocado.ccs.scripts:DockerScriptsLibrary) will need to be updated as follows:

1. [Locate](https://euw1-ocssourcegraph.eu-west-1.aws.shd.dev.lastmile.com/search?q=@Grab+com.ocado.ccs.scripts:DockerScriptsLibrary) the groovy script(s) which depends on your image
2. Make sure the script uses DockerScriptsLibrary version 1.60 or newer
3. Find the `ComposeProject.create(projectName, composeFilesList, imagesMap, <optional args...>)` method call in the script.
4. Remove entry of your project from the imagesMap argument.
5. Go through composeFilesList, find mentions of your project and inline the contents of the `docker-compose-common.yml` in place of those references:
    * update docker-compose version: declaration '2.0' to '2.4'
    * inline the referenced service (while also inlining the services this service inherits from if any), not the entire file; you can get an inlined service definition by starting the groovy script with an image with docker-compose-common.yml still existing
    * modify the inlined contents:
    * any `expose` entries should be moved to the `EXPOSE` declaration in Dockerfile of the image (this should be already done)
    * contents of `volumes` declaration should be replaced with `${DEV_IMAGES_WORKDIR}/<projectname>/logs:/logs` and `user: "root"` (to enable the entrypoint to set correct access rights flags for the image)
    * contents of `env` declaration should preferably replaced with `env: MODE: <mode>`, see [environment configuration](#dockerizing-applications_configuring-applications_environment-configuration), as the default env variables for a mode should be part of the image
    * `image:` declaration should be replaced with `image: ${servicename_IMAGE_TAG:-internal.docker.ocean.tech.lastmile.com/ocean_cfc/<name>:ci-master-latest}`
    * `links` declaration should be replaced with `depends_on:`
        * `links: -db -activemq` should be replaced with: (makes docker-compose start services in order)
```
depends_on: # wait for db/activemq to fully recover before starting the services depending on them
      db:
        condition: service_healthy
      activemq:
        condition: service_healthy
```

NOTE: depends_on with condition syntax is not supported since version 3 of docker compose config file, in which case some other ideas could be used to ensure waiting for ActiveMQ and database to startup. See the discussion at https://ocado.slack.com/archives/CFHLU4X8E/p1553880711016800?thread_ts=1553770983.001200&cid=CFHLU4X8E