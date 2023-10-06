# Docker Notes
A cheat-sheets and quick reference guide for docker CLI commands and some of docker concepts

## Docker CLI
Show commands & management commands, versions and infos
```bash
$ docker #Display Docker commands
$ docker version #Show the Docker version information
$ docker info #Display system-wide information
```

## Containers
##### Create and run a new container from an image
```bash
docker run --name NAME --rm -itd -p PORTS  --net=<custom_net> --ip=IP image COMMAND
```
**Options & Tags**<br>
`--name` : The name of the container, should be unique, a tag of the container can be specified (can granted the _uniqueness_ of the container) `name:tag`<br>

`--rm` : Remove the container when it is stopped <br>

`-i` : Keep standard output stream open<br>
`-t` : Run the container in a pseudo-TTY [interactive mode]<br>
`-d` : Run the container in the background [detach mode]<br>

`-p PORTS` : Publish or expose ports<br>
- `-p 80` : Equivalent to `-p 80:80`
- `-p 80:80` :  Binds port `80` of the container to  port `80`  of the host machine , accessible externally
- `-p 127.0.0.1:80:80`: Binds port `80` of the container to  port `80` on `127.0.0.1` of the host machine (only accessible  by the host machine)
- `--expose 80` : Exposes port 80 of the container without publishing the port to the host system's interfaces (used mostly for communication between containers)<br>

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


## Images
##### Download an image from a registry (Such as Docker Hub)
```bash
$ docker pull image
```
**Options & Tags**<br>
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
**Options & Tags**<br>
- `.` : The `PATH`  specifies where to find the files for the _context_ of the build on the Docker daemon.
- `-t NAME` : The name of the image to build, , a tag of the image can be specified `image:tag`.
- `-f`  :  Specify the Dockerfile name from which to build the image. by default if no dockerfile is specified , docker daemon will search for a file name _Dockerfile_ in the build _context_ (Default is `PATH/Dockerfile`)
-  `--rm`: Remove intermediate containers after a successful build

you can find here [the complete list of options](https://docs.docker.com/engine/reference/commandline/build/#options) you can use with the _docker build_ command

Note : All the files in the local directory (`PATH`) get `tar'd` and sent to the Docker daemon.You can exclude files and directories by adding a `.dockerignore` file to the local directory, This helps to avoid unnecessarily sending large or sensitive files and directories to the daemon. [check-here-to-read-about-dockerignore](dockerfile.readme)

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

`docker rmi $(docker images -aq)`: Remove all images
`docker rmi $(docker images -f "dangling=true" -q)`:  Remove all _dangling_ images. (Docker images that are untagged and unused layers such the intermediate images)

`docker image prune [ID/NAME]` : by default this command has same behavior as above command (removing  all _dangling_ images)
- `-a`:  If this flag is specified, will also remove all images not referenced by any container.
