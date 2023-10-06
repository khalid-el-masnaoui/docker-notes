# Docker Notes
A cheat-sheets and quick reference guide for docker CLI and some of docker concepts

### Docker CLI
Show commands & management commands, versions and infos
```bash
$ docker #Display Docker commands
$ docker version #Show the Docker version information
$ docker info #Display system-wide information
```

### Containers & Images
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

`image` :  The image from which the container will be created and run, a tag of the image can be specified `image:tag` . The image can be locally stored Docker images. If you use an image that is not on your system, the software pulls it from the online registry.<br>

`COMMAND`: Used to execute a command inside the container , example `sh -c "echo hello-world"`<br>

**Example**
```bash
$ docker run --name nginx:malidkha --rm -itd -p 80:80  --net=malidkha-network --ip='10.7.0.2' nginx:1.2 bash -c "echo hello-there"
```

##### Listing , start, stop and other docker container commands
```bash
$ docker ps #Display all running docker containers
$ docker container start [ID/NAME] #Start one or more containers by NAME or ID
$ docker container stop [ID/NAME] #Stop one or more containers by NAME or ID
$ docker restart [ID/NAME] #Restart one or more containers by NAME or ID
$ docker container rm [ID/NAME] #Remove one or more stppped containers by NAME or ID
$ docker rename NAME-OLD NAME-NEW #Rename a container
```
**Options & Additional commands**<br>

`docker ps` : Equivalent to `docker container ls` 
- `-a` :   List all containers (running & stopped)
- `-q` : Return only the ID of each container
- `-l` : Return the last container   

`docker stop $(docker ps -aq)` : Stop all  containers
`docker kill [ID/NAME]`: Kill  one or more  containers by NAME or ID, stop command send _SIGTERM_ signal, while kill  sends the _SIGKILL_ signal

`docker rm $(docker ps/image -aq)`: Remove all containers
- `-f`: By default remove command , remove the container if it is already stopped , this flag forces the container to be removed even if it is running
- `-v`: Remove anonymous volumes associated with the container  [check-volume-section](#Volumes) 
