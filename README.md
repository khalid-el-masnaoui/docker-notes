# Docker Notes
A cheat-sheets and quick reference guide for docker CLI commands and some of docker concepts

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
[discover-docker-images-and-containers-concepts-here](docker.readme)
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

**_Dockerfiles_** are text files that list instructions for the Docker daemon to follow when building a container image.  [check-here-to-read-about-Dockerfiles](dockerfiles.md)
```bash
$ docker build . -t NAME -f DOCKERFILE --rm
```
**Options & Flags**<br>
- `.` : The `PATH`  specifies where to find the files for the _context_ of the build on the Docker daemon.
- `-t NAME` : The name of the image to build, , a tag of the image can be specified `image:tag`.
- `-f`  :  Specify the Dockerfile name from which to build the image. by default if no dockerfile is specified , docker daemon will search for a file name _Dockerfile_ in the build _context_ (Default is `PATH/Dockerfile`)
-  `--rm`: Remove intermediate containers after a successful build

you can find here [the complete list of options](https://docs.docker.com/engine/reference/commandline/build/#options) you can use with the _docker build_ command

**Note** : All the files in the local directory (`PATH`) get `tar'd` and sent to the Docker daemon.You can exclude files and directories by adding a `.dockerignore` file to the local directory, This helps to avoid unnecessarily sending large or sensitive files and directories to the daemon. [check-here-to-read-about-dockerignore](dockerfile.readme)

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
[discover-docker-images-and-containers-concepts-here](docker.readme)
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

`--net` : Connect a container to a network [check-network-section](#Networking) <br>
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
- `-v`: Remove anonymous volumes associated with the container  [check-volume-section](#Volumes) 

**Access & Run shell commands Inside the container**<br>
```bash
$ docker container exec -it [NAME] COMMAND

#examples
$ docker container exec -it [NAME] bash #Access the bash shell of the container
$ docker container exec -it [NAME] touch hello.txt #Create hello.txt file inside the working directory of the container
```

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

**Note** : Though do note here that the above command is only functional for containers that are started with the `json-file`  or `journald` logging driver.

**Note 2** : As a default, Docker uses the `json-file` logging driver, which caches container logs as JSON internally. [Read more about this logging driver](https://docs.docker.com/config/containers/logging/json-file/)

**Example**

```bash
$ docker logs -f --since=10m TEST #Show the logs for the container "TEST"  for the past 10 minutes
```

##### Docker Logs Location

Docker, by default, captures the standard output (and standard error) of all your containers and writes them in files using the JSON format. This is achieved using JSON File logging driver or `json-file`. These logs are by default stored at container-specific locations under `/var/lib/docker` filesystem.

```bash
/var/lib/docker/containers/container_id>/<container_id>-json.log
```

## Volumes :card_file_box:
All the changes inside the container are lost when the container stops. If we want to keep data between runs or to share data between different containers, Docker volumes and bind mounts come into play.

#####  The Docker File System

A docker container runs the software stack defined in an [image](#images-framed_picture). Images are made of a set of read-only layers that work on a file system called the Union File System. When we start a new container, Docker adds a read-write layer on the top of the image layers allowing the container to run as though on a standard Linux file system.

So, any file change inside the container creates a working copy in the read-write layer. However, **when the container is stopped or deleted, that read-write layer is lost**.


![](https://www.baeldung.com/wp-content/uploads/2021/01/layers.png)

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

To use a volume , we can either specify the volume in the [dockerfile](#dockerfile.md) or the [docker-compose-file](#docker-compose.md) , as well as starting the container with a volume in the command.

###### Starting a Container with a Volume

**Using -v (--volume)**

```bash
$ docker run -v data-volume:/var/opt/project(:ro) TEST #Named VOLUME
$ docker run -v /var/opt/project(:ro) TEST #Anonymous VOLUME
$ docker run -v ./src/data:/var/opt/project(:ro) TEST #BIND MOUNT
```

- _1st example_  : The above example is starting a container TEST, and attaching to it a volume `data-volume`  to  persist data inside docker container folder `/var/opt/project`.

- _2nd example_  : The above example is starting a container TEST, and attaching to it an anonymous volume (Docker gives the volume a random name that is guaranteed to be unique within a given Docker host) to persist data inside docker container folder `/var/opt/project`. 

-  _3rd example_  : The above example is starting a container TEST, and attaching to it source directory on the host `./src/data` to persist data inside docker container folder `/var/opt/project`.

The option `(:ro)` is optional specifying that the data is read-only by the container 

**Using –mount**

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

##### Share data volumes with `--volumes-from`

We should note that attaching a volume to a container creates a long-term connection between the container and the volume. Even when the container has exited, the relationship still exists. This allows us to use a container that has exited as a template for mounting the same set of volumes to a new one.

**example**
We start a container TEST with volume  mounted to `/var/opt/project`
```bash
 $ docker run -v /var/opt/project TEST #Named VOLUME

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