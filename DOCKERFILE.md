
## Dockerfile Notes

Notes about _Dockerfile_, a complete Dockerfile instruction reference and some best practices.


## What is a Dockerfile

###### Introduction

**_Dockerfiles_** are text files that list instructions for the Docker daemon to follow when building a container image

When you execute the `docker build` command, the lines in your Dockerfile are processed sequentially to assemble your image (in a _layered_ fashion).

The basic Dockerfile syntax looks as follows:

```dockerfile
# Comment
INSTRUCTION arguments
```

Instructions are keywords that apply a specific action to your image, such as copying a file from your working directory, or executing a command within the image.

By convention, instructions are usually written in uppercase. This is not a requirement of the Dockerfile parser, but it makes it easier to see which lines contain a new instruction.

Lines starting with a `#` character are interpreted as comments. They’ll be ignored by the parser, so you can use them to document your Dockerfile.

###### Example

```dockerfile
FROM ubuntu:22.04
COPY . /app
RUN make /app
CMD python /app/app.py
```

In the example above, each instruction creates one layer:

- `FROM` creates a layer from the `ubuntu:22.04` Docker image.
- `COPY` adds files from your Docker client's current directory.
- `RUN` builds your application with `make`.
- `CMD` specifies what command to run within the container.

**Note** : When you run an image and generate a container, you add a new _writable layer_, also called the container layer, on top of the underlying layers. All changes made to the running container, such as writing new files, modifying existing files, and deleting files, are written to this writable container layer (writable layer is not persistent,  [read about that here](README.md#volumes-card_file_box) ).


## Dockerfile Instructions Reference

Docker supports over 15 different Dockerfile instructions for adding content to your image and setting configuration parameters. We will discuss all of them.

Docker runs instructions in a `Dockerfile` in order. A `Dockerfile` **must begin with a `FROM` instruction**. (`ARG` is the only instruction that may precede `FROM` in the `Dockerfile`.)

##### List of the instruction

```dockerfile 
FROM nginx:latest
```
- We use `FROM` to specify the base image we want to start from. (base image and also alias for _multi-stage_ build)
 - The `tag` values are optional. If you omit it, the builder assumes a `latest` tag by default. The builder returns an error if it cannot find the `tag` value.

```dockerfile 
MAINTAINER malidkha
```
- Set the Author field of the generated images.

```dockerfile 
LABEL name="malidkha"
LABEL email="malidkha.elmasnaoui@gmail.com"
LABEL description="This is a Dockerfile reference"
```
-  Labels are key-value pairs and simply adds custom metadata to your Docker Images (such as version , release-date ..)

```dockerfile 
#shell form
RUN apt-get update -y && apt-get install -y procps
RUN chown -R malikdha:malikdha /var/log/nginx && \
    chown -R malikdha:malikdha /etc/nginx

#exec form
RUN ["apt-get", "update", "-y"]
```
-  `RUN` is used to run commands during the image build process. (installing packages using apt , npm , composer ..)
- **_Shell form_** : `RUN COMMAND` (by default is `/bin/sh -c`)
- **Exec form**  : `RUN ["<executable>", "<param1>", "<param2>"]`
- Change shell In which to run the command : `RUN ["/bin/bash", "-c", "echo hello from bash"]`

```dockerfile 
#shell form
CMD nginx -g daemon off

#exec form
CMD ["/usr/sbin/nginx", "-g", "daemon off;"]
```
-  `CMD`:  Executes a command within a running container
- **_Shell form_** : `CMD <command> <param1> <param2` (by default is `/bin/sh -c`)
- **Exec form**  : `CMD ["executable", "param1", "param2"]`
- Change shell In which to execute the command : `CMD ["/bin/bash", "-c", "echo hello from bash"]`
- `CMD` Runs when the containers starts , only one can be executed, the _last one_
- We can override the `CMD` instruction using the `docker run` command. If we specify a `CMD` in our Dockerfile and one on the `docker run` command line, then the command line will _override_ the Dockerfile ’s `CMD` instruction.


```dockerfile 
#shell form
ENTRYPOINT nginx -g daemon off

#exec form
ENTRYPOINT ["/usr/sbin/nginx", "-g", "daemon off;"]
```
-  `ENTRYPOINT`:  Specifies the commands that will execute when the Docker container starts. If you don’t specify any `ENTRYPOINT`, it defaults to `/bin/sh -c`
- **_Shell form_** : `ENTRYPOINT <command> <param1> <param2` (by default is `/bin/sh -c`)
- **Exec form**  : `ENTRYPOINT ["executable", "param1", "param2"]`
- Change shell In which to execute the command : `ENTRYPOINT ["/bin/bash", "-c", "echo hello from bash"]`
- `ENTRYPOINT` Sets default parameters that cannot be overridden while executing Docker containers with CLI parameters(cli will be appended as argumenets).
- `ENTRYPOINT` can be overridden during running docker using `--entrypoint COMMAND`flag white `docker run`command

```dockerfile     
ENTRYPOINT ["/usr/sbin/nginx"]
CMD ["-g", "daemon off;"]
```

- We can use both `CMD` and `ENTRYPOINT` ---> `ENTRYPOINT` will be executed first followed by `CMD` as argument

```dockerfile
SHELL ["executable", "parameters"]

#change default shell
SHELL ["/bin/bash", "-c"]
```

- `SHELL` instruction allows the default shell used for the shell form of commands to be overridden. The default shell on Linux is `["/bin/sh", "-c"]`.
- The `SHELL` instruction can appear multiple times. Each SHELL instruction overrides all previous SHELL instructions, and affects all subsequent instructions.
- All instructions that use a shell , or that come after `SHELL`instruction, will use the new shell.

```dockerfile
EXPOSE 80/tcp
```

- `EXPOSE`is Used to tell Docker that the container listens on the specified network ports at runtime.Default is _TCP_ if the protocol is not specified.
- This instruction won't open any port, as we can't open a port on a Docker Image, it is just to let docker know that the container will listen to the specified port at runtime

```dockerfile
#ARG INstruction
ARG NAME=malikdha
ARG NAME2

#ENV Insruction
ENV NAME malidkha
ENV NAME2=malidkha
```

- `ARG` Defines build-time variables using key-value pairs. However, these ARG variables will not be accessible when the container is running.
-  The `ARG` instruction defines a variable that users can pass at build-time to the builder with the `docker build`command using the `--build-arg <varname>=<value>` flag

- `ENV` Sets environment variables within the image, making them accessible both during the build process and while the container is running.
- This instruction won't open any port, as we can't open a port on a Docker Image, it is just to let docker know that the container will listen to the specified port at runtime


```dockerfile
#COPY Instruction
COPY /path/on/host/ /path/on/container/

#ADD Instruction
ADD /path/on/host/ /path/on/container/
```

- `COPY`is used to copy a file or folder from the host system into the docker image.
- We can copy the file or folder , while setting its ownership on the image using `COPY --chown=<user>:<group>`
- `ADD` is an _advanced form of `COPY`_, Copies new files, directories, or remote file URLs from  the host filesystem  into the docker image.
- `ADD` also supports 2 additional sources. First, you can use a _URL_ instead of a local file / directory. Secondly, you can extract a _tar file_ from the source directly into the destination.
- A valid use case for `ADD` is when you want to extract a local tar file into a specific directory in your Docker image.
- If you’re copying in local files to your Docker image, always use `COPY` because it’s more explicit.

```dockerfile
WORKDIR /var/www/html
```

- `WORKDIR`is Used to set the current working directory.
- The `WORKDIR` instruction can be used multiple times in a `Dockerfile`. If a relative path is provided, it will be relative to the path of the previous `WORKDIR` instruction. For example:

```dockerfile
USER malidkha
```

- `USER`sets the user name or _UID_ to use when running the image and for any `RUN`, `CMD`, `ENTRYPOINT` and any instruction instructions that follow it in the `Dockerfile`.
- The `WORKDIR` instruction can be used multiple times in a `Dockerfile`. If a relative path is provided, it will be relative to the path of the previous `WORKDIR` instruction. For example:
- If the User does not exit on the container you may want to add the user first : `RUN groupadd -f malidkha && (id -u malidkha &> /dev/null || useradd -G malidkha malidkha -D)`


```dockerfile
VOLUME ["/var/log/nginx", "/var/log/ph"]
VOLUME /var/log/nginx /var/log/php
```

- `VOLUME` creates a mount point with the specified name and marks it as holding externally mounted volumes from native host or other containers.


```dockerfile
ONBUILD RUN composer install --optimize-autoloader --no-dev
```

- `ONBUILD`executes after the current Dockerfile build completes. 
- The `ONBUILD` instruction may not trigger `FROM`, `MAINTAINER`, or `ONBUILD` instructions.





