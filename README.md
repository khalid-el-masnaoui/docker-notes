# Docker Notes
A cheat-sheets and quick but relatively-detailed reference guide for docker CLI commands and some of docker concepts

This repository contained a complete _**set**_ of DOCKER NOTES organized as follow:

- [Docker Architecture](ARCHITECTURE.md) 
- [Docker CLI ](README.md) (_current file_)
- [Dockerfile](DOCKERFILE.md)
- [Docker Compose](DOCKER-COMPOSE.md)
- [Docker Best Practices](BEST-PRACTICES.md)

# Table Of Contents

- **[Docker CLI](#docker-cli-whale)**
- **[Images](#images-framed_picture)**
	- **[Download an image from a registry](#download-an-image-from-a-registry-such-as-docker-hub)**
	- **[Build An image locally from a Dockerfile](#build-an-image-locally-from-a-dockerfile)**
	- **[List , tag, remove and other docker image commands](#list--tag-remove-and-other-docker-image-commands)**
- **[Containers](#containers-ship)**
	- **[Create and run a new container from an image](#create-and-run-a-new-container-from-an-image)**
	- **[List , start, stop and other docker container commands](#list--start-stop-and-other-docker-container-commands)**
	- **[Access & Run shell commands Inside the container](#access---run-shell-commands-inside-the-container)**
- **[Docker Logs](#docker-logs-spiral_notepad)**
	- **[Docker Logs Commands](#docker-logs-commands)**
	- **[Docker Logs Location](#docker-logs-location)**
	- **[Docker Logs Rotation Configuration](#docker-logs-rotation-configuration)**
- **[Volumes](#volumes-card_file_box)**
    - **[The Docker File System](#the-docker-file-system)**
    - **[Creating and managing a volume](#creating-and-managing-a-volume)**
    - **[Usage](#usage)**
		- **[Starting a Container with a Volume](#starting-a-container-with-a-volume)**
		- **[Using -v (--volume)](#using-v)**
		- **[Using –mount](#using-mount)**
    - **[What and When to use a specific volume type](#what-and-when-to-use-a-specific-volume-type)**
    - **[Share data volumes with --volumes-from](#share-data-volumes-with---volumes-from)**
- **[Networking](#networking-earth_africa)**
	- **[How it works](#how-it-works)**
	- **[Network drivers](#network-drivers)**
	- **[Creating and managing a network](#creating-and-managing-a-network)**
	-  **[Usage](#usage-1)**
		-  **[Connect a container to a network](#connect-a-container-to-a-network)**
- **[Monitoring and Manage docker containers](#monitoring-and-manage-docker-containers-oncoming_police_car)**
	- **[Introduction](#introduction)**
	- **[Monitor docker containers](#monitor-docker-containers)**
	- **[Manage docker containers resources](#manage-docker-containers-resources)**
		-  **[Configure System to Enable Limiting Resources](#manage-res-configs)**
		-  **[Limit Docker Container Memory Access](#limit-memory)**
		-  **[Limit Docker Container CPU Usage](#limit-cpu)**

## Docker CLI :whale:
Show commands & management commands, versions and infos
```bash
$ docker #Display Docker commands
$ docker version #Show the Docker version information
$ docker info #Display system-wide information
```

## Images :framed_picture:
Images are read-only templates containing instructions for creating a container. A Docker image creates containers to run on the Docker platform.
Think of an image like a blueprint or snapshot of what will be in a container when it runs.
[discover-docker-images-and-containers-concepts-here](ARCHITECTURE.md)
##### Download an image from a registry (Such as Docker Hub)
```bash
$ docker pull image
```
**Options & Flags**<br>
`image` : The name of the image to download, , a tag of the image can be specified `image:tag`.  If no tag is provided, Docker Engine uses the `:latest` tag as a default.<br>
- `-a`  :  Download all tagged images in the repository
-  `--platform`: Set platform if server is multi-platform capable
you can find here [the complete list of options](https://docs.docker.com/engine/reference/commandline/pull/#options) you can use with the _docker pull_ command

**Example**
```bash
$ docker pull nginx #Download nginx latest image
$ docker pull nginx:alpine #Download alpine nginx image
```
##### Build An image locally from a Dockerfile

**_Dockerfiles_** are text files that list instructions for the Docker daemon to follow when building a container image.  [check-here-to-read-about-Dockerfiles](DOCKERFILE.md)
```bash
$ docker build . -t NAME -f DOCKERFILE --rm
```
**Options & Flags**<br>
- `.` : The `PATH`  specifies where to find the files for the _context_ of the build on the Docker daemon.
- `-t NAME` : The name of the image to build, , a tag of the image can be specified `image:tag`.
- `-f`  :  Specify the Dockerfile name from which to build the image. by default if no dockerfile is specified , docker daemon will search for a file name _Dockerfile_ in the build _context_ (Default is `PATH/Dockerfile`)
-  `--rm`: Remove intermediate containers after a successful build

you can find here [the complete list of options](https://docs.docker.com/engine/reference/commandline/build/#options) you can use with the _docker build_ command

**Note** : All the files in the local directory (`PATH`) get `tar'd` and sent to the Docker daemon.You can exclude files and directories by adding a `.dockerignore` file to the local directory, This helps to avoid unnecessarily sending large or sensitive files and directories to the daemon. [check-here-to-read-about-dockerignore](DOCKERFILE.md#dockerignore-file)

**Example**
```bash
$ docker build . -t custom-image -f Dockerfile.dev
```

##### List , tag, remove and other docker image commands
```bash
$ docker images #Display all images
$ docker rmi [ID/NAME] #Remove one or more images by NAME or ID
$ docker image prune #Remove unused images
$ docker tag [NAME/ID] NAME:TAG #Create a tag NAME:TAG image that refers to the source image by NAME or ID 
$ docker history [NAME/ID] #Show the history of an image 
$ docker image inspect [ID/NAME]  #Display detailed information on one or more images such as Exposed Ports, Environement variables..
```
**Options & Additional commands**<br>

`docker images` : Display all parent images 
- `-a` :   List all images (parents & intermediates)
- `-q` : Return only the ID of each image
- `-l` : Return the last container   

`docker rmi [ID/NAME]` : Equivalent to `docker image rm [ID/NAME]`

`docker rmi $(docker images -aq)`: Remove all images<br>
`docker rmi $(docker images -f "dangling=true" -q)`:  Remove all _dangling_ images. (Docker images that are untagged and unused layers such the intermediate images)

`docker image prune [ID/NAME]` : by default this command has same behavior as above command (removing  all _dangling_ images)
- `-a`:  If this flag is specified, will also remove all images not referenced by any container.

## Containers :ship:
A container is an isolated place where an application runs without affecting the rest of the system and without the system impacting the application. 
[discover-docker-images-and-containers-concepts-here](ARCHITECTURE.md)
##### Create and run a new container from an image
```bash
docker run --name NAME --rm -itd -p PORTS  --net=<custom_net> --ip=IP image COMMAND
```
**Options & Flags**<br>
`--name` : The name of the container, should be unique, a tag of the container can be specified (can granted the _uniqueness_ of the container) `name:tag`<br>

`--rm` : Remove the container when it is stopped <br>

`-i` : Keep standard output stream open<br>
`-t` : Run the container in a pseudo-TTY [interactive mode]<br>
`-d` : Run the container in the background [detach mode]<br>

`-p PORTS` : Publish or expose ports<br>
- `-p 80` : Equivalent to `-p 80:80`
- `-p 80:80` :  Binds port `80` of the container to  port `80`  of the host machine , accessible externally
- `-p 127.0.0.1:80:80`: Binds port `80` of the container to  port `80` on `127.0.0.1` of the host machine (only accessible  by the host machine)
- `--expose 80` : Exposes port 80 of the container without publishing the port to the host system's interfaces<br>

`--net` : Connect a container to a network [check-network-section](#networking-earth_africa) <br>
`--ip`:  Assign a static IP to containers  (you must specify subnet block for the network)<br>

`image` :  The image from which the container will be created and run, a tag of the image can be specified `image:tag` . The image can be locally stored Docker images (locally built). If you use an image that is not on your system, the software pulls it from the online registry.<br>

`COMMAND`: Used to execute a command inside the container , example `sh -c "echo hello-world"`,  `bash`this will grant you access the bash shell inside the container<br>
you can find here [the complete list of options](https://docs.docker.com/engine/reference/commandline/run/#options) you can use with the _docker run_ command

**Example**
```bash
$ docker run --name nginx:malidkha --rm -itd -p 80:80  --net=malidkha-network --ip='10.7.0.2' nginx:1.2 bash -c "echo hello-there"
```

##### List , start, stop and other docker container commands
```bash
$ docker ps #Display all running docker containers
$ docker container start [ID/NAME] #Start one or more containers by NAME or ID
$ docker container stop [ID/NAME] #Stop one or more containers by NAME or ID
$ docker restart [ID/NAME] #Restart one or more containers by NAME or ID
$ docker rm [ID/NAME] #Remove one or more stppped containers by NAME or ID
$ docker container prune #Removes all stopped containers
$ docker rename NAME-OLD NAME-NEW #Rename a container
$ docker container inspect [ID/NAME]  #Display detailed information on one or more containers such as connected Networks settings, container IP,  volumes , Port Bindings ...
$ docker attach  [ID/NAME] #Attach to a running container and view its output

# copy files/directories between host and container
$ docker cp [NAME]:/PATH/ /local/path  #container -> host
$ docker cp /local/path [NAME]:/path/  #host -> container
```
**Options & Additional commands**<br>

`docker ps` : Equivalent to `docker container ls` 
- `-a` :   List all containers (running & stopped)
- `-q` : Return only the ID of each container
- `-l` : Return the last container   

`docker stop $(docker ps -aq)` : Stop all  containers
`docker kill [ID/NAME]`: Kill  one or more  containers by NAME or ID, stop command sends _SIGTERM_ signal, while kill  sends the _SIGKILL_ signal

`docker rm $(docker ps -aq)`: Remove all containers
- `-f`: By default remove command , remove the container if it is already stopped , this flag forces the container to be removed even if it is running
- `-v`: Remove anonymous volumes associated with the container  [check-volume-section](#volumes-card_file_box) 

##### Access & Run shell commands Inside the container
```bash
$ docker container exec -it [NAME] COMMAND

#examples
$ docker container exec -it [NAME] bash #Access the bash shell of the container
$ docker container exec -it [NAME] touch hello.txt #Create hello.txt file inside the working directory of the container
```

**Options & Flags** 

- `-w [DIR]` :   Working directory inside the container
- `-u [UserName/UID]` :Username or UID under which the command is executed
- `-e KEY=VALUE` : Set environment variables

you can find here [the complete list of options](https://docs.docker.com/engine/reference/commandline/exec/#options) you can use with _docker exec_ command 

## Docker Logs :spiral_notepad:

Like any other modern software, logging events and messages like warnings and errors is an inherent part of the Docker platform, which allows you to debug your applications and production issues.

##### Docker Logs Commands
```bash
$ docker logs [OPTIONS] [NAME/ID] #Fetch logs of a container by NAME or ID
```

**Options & Flags**

`docker logs`: Equivalent to `docker container logs`

- `--details` : Show extra details provided to logs.
- `--follow` or  `-f` :  Follow log output
- `--since`: Show logs since timestamp (e.g. 2021-08-28T15:23:37Z) or relative (e.g. 56m for 56 minutes)
- `--tail` or `-n` : all	Number of lines to show from the end of the logs
- `--timestamps` or `-t` : Show timestamps
- `--until` : Show logs before a timestamp (e.g. 2021-08-28T15:23:37Z) or relative (e.g. 56m for 56 minutes)

`docker logs -f [NAME/ID] &> malidkha.log &` : Redirect all live logs to a specific file. (background task)


**Note** : Though do note here that the above command is only functional for containers that are started with the `json-file`  or `journald` logging driver.

**Note 2** : As a default, Docker uses the `json-file` logging driver, which caches container logs as JSON internally (On the host). [Read more about this logging driver](https://docs.docker.com/config/containers/logging/json-file/)

**Example**

```bash
$ docker logs -f --since=10m TEST #Show the logs for the container "TEST"  for the past 10 minutes
```

##### Docker Logs Location

Docker, by default, captures the standard output (and standard error) of all your containers and writes them in files using the JSON format. This is achieved using JSON File logging driver or `json-file`. These logs are by default stored at container-specific locations under `/var/lib/docker` filesystem.

```bash
/var/lib/docker/containers/container_id>/<container_id>-json.log
```

##### Docker Logs Rotation Configuration

we can configure logs rotation when running a container with `docker run`command or via a global docker daemon setting

```bash
docker run --log-opt max-size=1k --log-opt max-file=5 <image-id/name>
```

Add the following code to  `/etc/docker/daemon.json` (global settings)
```bash
	{
	  "log-driver": "json-file",
		"log-opts": {
			"max-size": "1k",
			"max-file": "5"
		}
	  }

#Save the file and restart docker
$ systemctl restart docker
```

-  `max-size`: The maximum size of the log before it is rolled. A positive integer plus a modifier representing the unit of measure (k, m, or g)
-  `max-file`: The maximum number of log files that can be present. If rolling the logs creates excess files, the oldest file is removed. A positive integer. Defaults to 5.
## Volumes :card_file_box:

All the changes inside the container are lost when the container stops. If we want to keep data between runs or to share data between different containers, Docker volumes and bind mounts come into play.

#####  The Docker File System

A docker container runs the software stack defined in an [image](#images-framed_picture). Images are made of a set of read-only layers that work on a file system called the Union File System. When we start a new container, Docker adds a read-write layer on the top of the image layers allowing the container to run as though on a standard Linux file system.

So, any file change inside the container creates a working copy in the read-write layer. However, **when the container is stopped or deleted, that read-write layer is lost**.

<p align="center">
<img src="https://www.baeldung.com/wp-content/uploads/2021/01/layers.png"/>
</p>

When it comes to storing persistent docker container data , we have three options **Volumes**,  **Bind Mounts** and **Tmpfs mounts**

**Volumes** : Volumes are the preferred mechanism for persisting data generated by and used by Docker containers. A bind mount uses the host file system, but _Docker volume are native to Docker_.

**Bind Mounts** : Docker has offered this feature ever since its inception. There is less functionality available with binding mounts compared to volumes. Bind mounts mount files and directories from the host machine into the container.

 **Tmpfs mounts**: Stored in the host system's memory only, and are never written to the host system's filesystem (not persisted on disk).


<p align="center">
<img src="https://docs.docker.com/storage/images/types-of-mounts-bind.png"/>
</p>

**Note**:  Volumes are the preferred mechanism for persisting docker containers data . The volumes are stored in `/var/lib/docker/volumes`.

##### Creating and managing a volume

```bash
$ docker volume create NAME   #Create volume NAME ,if a name is not specified, Docker generates a random name.
$ docker volume ls            #Check volumes list
$ docker volume inspect NAME  #Shows some information like the mount-point, mount type...
$ docker volume rm NAME       #Remove a volume
$ docker volume prune NAME    #Remove all unused volumes and free up space:
```

**Options & Flags**

`docker volume create NAME`: will create a named volume,If a name is not specified, Docker generates a random name.
- Using this command , we can  specify other information about our created volume. you can find here [the complete list of options](https://docs.docker.com/engine/reference/commandline/volume_create/#options) you can use with this command


`docker volume ls`: 
- `-f`: Filer the list of volumes
	- `-f name=dataVolume` : Return the volume names _dataVolume_
	- `-f dangling=true` : Returns the volume names that are not connected to any containers
- `-q`: Return only the volume names

`docker volume rm $(docker volume ls -f dangling=true) `: Remove all dangling volumes

##### Usage

To use a volume , we can either specify the volume in the [docker-compose-file](#DOCKER-COMPOSE.md)  or  starting the container with a volume in the command.

###### Starting a Container with a Volume

<span id="using-v"><strong>Using -v (--volume)</strong></span>

```bash
$ docker run -v data-volume:/var/opt/project(:ro) TEST #Named VOLUME
$ docker run -v /var/opt/project(:ro) TEST #Anonymous VOLUME
$ docker run -v ./src/data:/var/opt/project(:ro) TEST #BIND MOUNT
```

- _1st example_  : The above example is starting a container TEST, and attaching to it a volume `data-volume`  to  persist data inside docker container folder `/var/opt/project`.

- _2nd example_  : The above example is starting a container TEST, and attaching to it an anonymous volume (Docker gives the volume a random name that is guaranteed to be unique within a given Docker host) to persist data inside docker container folder `/var/opt/project`. 

-  _3rd example_  : The above example is starting a container TEST, and attaching to it source directory on the host `./src/data` to persist data inside docker container folder `/var/opt/project`.

The option `(:ro)` is optional specifying that the data is read-only by the container 

<span id="using-mount"><strong>Using –mount</strong></span>

```bash
$ docker run --mount \ 
  type=volume,source=data-volume,destination=/var/opt/project,readonly NAME
```

- `type`: Specify the type of the volume
	- `volume`
	- `bind`
	- `tmpfs`
- `source`: The source of the mount (volume name or directory/file path for bind mount).
- `destination`: the path where the file or directory is mounted in the container.
- `readonly`: is optional specifying that the data is read-only by the container 
<br>

**Note**: It  is preferred  to use the more self-explanatory `–mount` option to specify the volume we wish to mount.

**Note 2** : When using the `-v` or `--volume` flag, Docker will automatically create the bind-mounted directory on your local machine if it doesn't exist.
When using the `--mount` flag, Docker will not create the bind-mounted directory on your local machine if it doesn't exist and generate an error.

**Note 3** :  `Bind-mounts` and `Named volumes` are persisted  and can be re-used within many containers and after re-run the container.
`Anonymous volumes` are not to be re-used , and after re-running the container are erased (stop and start commands will preserve the contents)

**Note 4** : Docker volumes is the recommended method for persisting Docker container data beyond the life of a container.
Bind mounts are another method, but are not recommended for most situations. Bind mounts can have dependencies on the underlying host file system directory, which may change over time.

##### What and When to use a specific volume type
###### Good use cases for Volumes

**Volumes are the preferred way to persist data** in Docker containers and services. Some use cases for volumes include:

- Sharing data among multiple running containers. If you don’t explicitly create it, a volume is created the first time it is mounted into a container. When that container stops or is removed, the **volume still exists**. Multiple containers can mount the same volume simultaneously, either read-write or read-only. Volumes are only removed when you explicitly remove them.
    
- When the Docker host is not guaranteed to have a given directory or file structure. Volumes help you decouple the configuration of the Docker host from the container runtime.
    
- When you want to store your container’s data on a remote host or a cloud provider, rather than locally.
    
- When you need to be able to **back up**, **restore**, or **migrate data** from one Docker host to another, volumes are a better choice. You can stop containers using the volume, then back up the volume’s directory (such as /var/lib/docker/volumes/).
    

###### Good use cases for Bind Mounts

In general, you should use volumes where possible. Bind mounts are appropriate for the following types of use case:

- Sharing configuration files from the host machine to containers. This is how Docker provides DNS resolution to containers by default, by mounting `/etc/resolv.conf` from the host machine into each container.
    
- Sharing source code or build artifacts between a development environment on the Docker host and a container. For instance, you may mount a `Maven target/` directory into a container, and each time you build the Maven project on the Docker host, the container gets access to the rebuilt artifacts.
    

If you use Docker for development this way, your production Dockerfile would copy the production-ready artifacts directly into the image, rather than relying on a bind mount.

- When the file or directory structure of the Docker host is guaranteed to be consistent with the bind mounts the containers require.

###### Good use cases for tmpfs mounts

`tmpfs mounts` are best used for cases when you do not want the data to persist either on the host machine or within the container. This may be for security reasons or to protect the performance of the container when your application needs to write a large volume of non-persistent state data.

##### Share data volumes with `--volumes-from`

We should note that attaching a volume to a container creates a long-term connection between the container and the volume. Even when the container has exited, the relationship still exists. This allows us to use a container that has exited as a template for mounting the same set of volumes to a new one.

**example**
We start a container TEST with volume  mounted to `/var/opt/project`
```bash
 $ docker run -v /var/opt/project TEST #Anonymous VOLUME

 #Later-on we stop the container and list it
 $ docker stop TEST
 $ docker ps -a
  | CONTAINER ID | IMAGE | COMMAND | CREATED | STATUS | PORTS | NAMES |
  | 4920602f8048 | bash  | "docker-entrypoint.s…" | 7 minutes ago Exited (0) | 7 minutes ago ...
```

We could run our next container, by copying the volumes used by this one:
```bash
$ docker run --volumes-from 4920 \
  bash:latest \
  bash -c "ls /var/opt/project"
```

**Note** : `--volumes-from`can be use in order to Back up, restore, or migrate data volumes.
In practice `--volumes-from` is usually used to link volumes between running containers (volume data migration).

## Networking :earth_africa:

Networking is about communication among processes, and Docker’s networking is no different. Docker networking is primarily used to establish communication between Docker containers and the outside world via the host machine where the Docker daemon is running.

<p align="center">
<img src="https://camo.githubusercontent.com/0caf43acba7beb411dca0af77c2d6f20e93ff0a19139c190382674efc3b1620f/68747470733a2f2f676f6c646d616e6e2e706c2f696d616765732f646f636b65722d6e6574776f726b2f6e6574776f726b2e706e67"/>
</p>

##### How it works
Docker uses your host’s network stack to implement its networking system. It works by manipulating **_iptables_** rules to route traffic to your containers. This also provides isolation between Docker networks and your host.
_iptables_ is the standard Linux packet filtering tool. Rules added to _iptables_ define how traffic is routed as it passes through your host’s network stack. Docker networks add filtering rules which direct matching  traffic to your container’s application. The rules are automatically configured, so you don’t need to manually interact with _iptables_.

Docker supports different types of networks via network drivers, each fit for certain use cases.

##### Network drivers
There are several default network drivers available in Docker and some can be installed with the help of plugins, Command to see the list of containers in Docker mentioned below.

1. **bridge:** The `bridge` network represents the `docker0` network present in all Docker installations. If you build a container without specifying the kind of driver, the container will only be created in the bridge network, which is the default network. 
	-  We can see this bridge as part of a host’s network stack by using the `ip addr show` command:
		```bash
		$ ip addr show

		docker0   Link encap:Ethernet  HWaddr 02:42:47:bc:3a:eb
		          inet addr:172.17.0.1  Bcast:0.0.0.0  Mask:255.255.0.0
		          inet6 addr: fe80::42:47ff:febc:3aeb/64 Scope:Link
		          UP BROADCAST RUNNING MULTICAST  MTU:9001  Metric:1
		          RX packets:17 errors:0 dropped:0 overruns:0 frame:0
		          TX packets:8 errors:0 dropped:0 overruns:0 carrier:0
		          collisions:0 txqueuelen:0
		          RX bytes:1100 (1.1 KB)  TX bytes:648 (648.0 B)
		```
	- 
1. **host:** The `host` network adds a container on the host’s network stack. As far as the network is concerned, _there is no isolation between the host machine and the container_. Containers will not have any IP address they will be directly created in the system network which will remove isolation between the docker host and containers.  For instance, if you run a container that runs a web server on port 80 using host networking, the web server is available on port 80 of the host machine.

2. **none:** The `none` network adds a container to a container-specific network stack. That container lacks a network interface. IP addresses won’t be assigned to containers. These containers are not accessible to us from the outside or from any other container. 

3. **overlay:**  The `overlay` network will enable the connection between multiple Docker daemons and make different Docker swarm services communicate with each other.

4. **ipvlan:** Users have complete control over both IPv4 and IPv6 addressing by using the IPvlan driver.
5. **macvlan:**  The `macvlan` driver makes it possible to assign MAC addresses to a container.

**Note** : When you install Docker, it creates **three networks** automatically: `bridge`, `none` and `host`.

##### Creating and managing a network

```bash
$ docker network create NAME   #Create network NAME
$ docker network ls            #Check networks list
$ docker network inspect NAME  #Shows some information such as network ips, gateway, connected containers...
$ docker network rm NAME       #Remove a network
$ docker network prune NAME    #Remove all unused networks
```

**Options & Flags**

`docker volume create NAME`: Creates a new network , if driver option is not set , will be defaulted to `bridge`
- `-d`:  Specify  Driver to manage the Network
- `--subnet`: Subnet in CIDR format that represents a network segment
- `--gateway` : IPv4 or IPv6 Gateway for the master subnet
-  You can find here [the complete list of options](https://docs.docker.com/engine/reference/commandline/network_create/#options) you can use with this command
-  Example : 
```bash
$ docker network create --subnet 10.7.0.0/16 --gateway 10.7.7.7 malidkha-network
```

 <h5><span id="usage-network">Usage</span></h5>

To use a network with a container , we can either specify the network in the [docker-compose-file](#DOCKER-COMPOSE.md) or connect a container to a network.

###### Connect a container to a network

```bash
$ docker network connect NAME <container-id/name> # Connect a running container to a network
$  docker run -itd --network=NAME <image-name> # Connect a container to a network when it starts
```

You can specify the IP address you want to be assigned to the container's interface.

```bash
$ docker network connect --ip 10.7.0.2 NAME <container-id/name>
$ docker run -itd --net=NAME --ip='10.7.0.2' <image-name>
```

 Use `docker network disconnect` to remove a container from the network.
```bash
$ docker network disconnect NAME <container-id/name>
```


**Note**: To verify the container is connected, use the `docker network inspect` command or `docker container inspect` command.

**Note 2** : One container can be connected to many networks, and One network can be attached to many containers.

**Note 3** : 
-  Two containers on same the default bridge network can communicate only by ip-addresses
-  Two containers on same the same bridge network (user-defined network) can communicate using both container names and ip-addresses.
 - If need to connect using container names and ip-addresses while on two different networks -- use `--link` (deprecated,  use user-defined networks instead)
 - Two containers on different networks can't connect via ip-addresses or container names( _iptables_ rules drop the packets)
 - Container on whatever network can connect with the host


## Monitoring and Manage docker containers :oncoming_police_car:

##### Introduction
Docker container monitoring is the process of tracking the performance, health, and resource utilization of applications running in Docker containers
Some of the standard metrics monitored in the Docker container are:

1. **Resource utilization metrics**: Metrics that refer to the resources allocated to and used by the Docker containers. Usage can indicate if the container is experiencing a workload too heavy, optimal, or light and helps troubleshoot issues with the application running inside the container. Some examples of these metrics include:
    - _CPU usage_: The percentage of CPU time being used by the container
    - _Memory usage_: The amount of memory being used
    - _Disk usage_: The amount of disk space being used
    - _Network usage_: The amount of network bandwidth being used
2. **Container status metrics**: These metrics refer to the status of the Docker containers to help track availability, overall health, and identify issues related to the container. Some examples of these metrics are:
    - _Container up-time_: The amount of time that the container has been running. High up-time can be an indicator of the stability and reliability of the container
    - _Container restarts_: The number of times that the container has been restarted. A high number of restarts can be an indicator of issues with the container or the application running inside it
    - _Container exits_: The number of times that the container has exited. High numbers of container exits can be an indicator of issues
    - _Status_: This metric refers to the status of the container itself (e.g. running, stopped, restarting)
3. **Application performance metrics**: Metrics that refer to the performance of the applications running inside Docker containers. Some examples are:
    - _Response time_: The time it takes for the containerized application to respond to a request. Tracking response time can help identify performance issues and ensure that the application is responsive to end users
    - _Request rate_: The number of requests being handled by the containerized application per second
    - _Error rate_: The number of errors encountered by the containerized application
4. **Host machine metrics**: Metrics that refer to the performance of the physical or virtual machine on which Docker is running. These metrics are essential to monitor because they can help identify issues related to the overall health and performance of the host machine, which can impact the performance and availability of the Docker containers running on the host. Some examples are:
    - _CPU usage_
    - _Memory usage_
    - _Disk usage_
    - _Network usage_
5. **Logs**: Logs provide a detailed record of the activities and events inside the container. It refers to tracking and analyzing the log output generated by a Docker container
6. **Processes**: Programs or tasks that are running inside the container

##### Monitor docker containers
There are several ways to monitor Docker containers in real time:
```bash
$ docker logs [NAME/ID] #Fetch logs of a container by NAME or ID
$ docker attach [NAME/ID] #Attach to a running container and view its output
$ docker top [NAME/ID] #Show running processes in a container
$ docker events [NAME/ID] #Get real-time events from the  container server such as restarts , starts ....
$ docker stats [NAME/ID] #Show Resource usage statisticsfor one or more containers (or all) such  CPU and memory usage 
$ docker system df -v #Show docker disk usage
$ docker port [NAME/ID] #Show mapped ports
$ docker diff [NAME/ID] #Show modified files
```

**Note** : You can find here [the complete list of the events](https://docs.docker.com/engine/reference/commandline/events/)

**Note 2** :Several third-party tools can monitor Docker containers in real time. These tools typically provide more advanced monitoring and visualization features, and integration with other IT systems and tools such _Prometheus_ and _cAdivsor_.


##### Manage docker containers resources

By default, Docker containers have access to the full RAM and CPU resources of the host. Leaving them to run with these default settings may lead to performance bottlenecks.
If you don’t limit Docker’s memory and CPU usage, Docker can use all the systems resources.

To limit a container resources , we can either specify that in the [docker-compose-file](#docker-compose.md)  or  using the `docker run` command,

<span id="manage-res-configs"><strong>Configure System to Enable Limiting Resources</strong></span>

Many of these features require your kernel to support Linux capabilities. To check for support, you can use the `docker info` command. If a capability is disabled in your kernel, you may see a warning at the end of the output like the following:

```bash 
WARNING: No swap limit support
```

If not enabled , enable limiting resources following those [instructions](https://docs.docker.com/engine/install/troubleshoot/#kernel-cgroup-swap-limit-capabilities)

<span id="limit-memory"><strong>Limit Docker Container Memory Access</strong></span>

There are several RAM limitations you can set for a Docker container. Some of them include:

- Configuring the maximum amount of memory a container can use.
- Defining the amount of memory a Docker container can swap to disk.
- Setting the soft limit for the amount of memory assigned to a container.

```bash
$ docker run -it --memory="[memory_limit]" <image-id/name> #Set Maximum Memory Access

$ docker run -it --memory="[memory_limit]" --memory-swap="[memory_limit]" <image-id/name> # Set Swap to Disk Memory Limit

$ docker run -it --memory="1g" --memory-reservation="750m" <image-id/name> #Set Soft Limit to Container Memory
```

`--memory="[memory_limit]"`:  The value of **`memory_limit`** should be a positive integer followed by the suffix **b**, **k**, **m**, or **g** (short for bytes, kilobytes, megabytes, or gigabytes). For example, to limit the container with 1 GB of RAM, add **`--memory="1g"`**.

`--memory-swap="[memory_limit]"`:  Using the **`swap`** option allows you to store data even after all RAM assigned to the container has been used up. It does this by ignoring the memory limitation and writing directly to the disk. Although this is a useful feature, it is not a recommended practice as it slows down performance. Value should be more than `--memory` (The swap includes the total amount of **non-swap memory** plus the amount of **swap memory** reserved as backup.)

`--memory-reservation="[memory_limit]"` : Limiting the memory usage of a container with **`--memory`** is essentially setting a hard limit that cannot be surpassed. Alternatively, you can set a soft limit (**`--memory-reservation`**) which warns when the container reaches the end of its assigned memory but doesn’t stop any of its services.

<span id="limit-cpu"><strong>Limit Docker Container CPU Usage</strong></span>

There are several ways to define how much CPU resources from the host machine you want to assign to containers.

```bash
$ docker run -it --cpus="1.0" <image-id/name> # The container will have access to only 1 CPU core

$ docker run -it --cpus-shares="700" <image-id/name> # The container will have a greater or lesser proportion of CPU cycles. By default, this is set to 1024.
```

Read more about CPU and MEMORY resources constraints [here](https://docs.docker.com/config/containers/resource_constraints/)