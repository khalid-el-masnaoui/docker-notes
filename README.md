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
Create and run a new container from an image
```bash
docker run --name NAME --rm -itd -p PORTS  --net=<custom_net> --ip=IP image COMMAND
```
**Options & Tags**<br>
`--name` : The name of the container, should be unique, a tag of the container can be specified (can granted the _uniqueness_ of the container) `name:tag`<br>

`--rm` : Remove the container when it is stopped <br>

`-i` : Keep standard output stream open
`-t` : Run the container in a pseudo-TTY [interactive mode]
`-d` : Run the container in the background [detach mode]<br>

**-p PORTS** : Publish or expose ports
`-p 80` : Equivalent to `-p 80:80`
`-p 80:80` :  Binds port `80` of the container to  port `80`  of the host machine , accessible externally
`-p 127.0.0.1:80:80`: Binds port `80` of the container to  port `80` on `127.0.0.1` of the host machine (only accessible  by the host machine)
`--expose 80` : Exposes port 80 of the container without publishing the port to the host system's interfaces (used mostly for communication between containers)<br>

`--net` : Connect a container to a network [check Network section](#Networking)
`--ip`:  Assign a static IP to containers  (you must specify subnet block for the network)<br>

`image` :  The image from which the container will be created and run, a tag of the image can be specified `image:tag` . The image can be locally stored Docker images. If you use an image that is not on your system, the software pulls it from the online registry.<br>

`COMMAND`: Used to execute a command inside the container , example `sh -c "echo hello-world"`<br>

**Example**
```bash
docker run --name nginx:malidkha --rm -itd -p 80:80  --net=malidkha-network --ip='10.7.0.2' nginx:1.2 bash -c "echo hello-there"
```