
## Dockerfile Notes

Notes about _Dockerfile_, a complete Dockerfile instruction reference and some best practices.


# Table Of Contents

- **[What is a Dockerfile?](#what-is-a-dockerfile)**
	- **[Introduction](#introduction)**
	- **[Example](#example)**
- **[Dockerfile Instructions Reference](#dockerfile-instructions-reference)**
	- **[List of the instructions](#list-of-the-instructions)**
- **[`.dockerignore` file](#dockerignore-file)**
	- **[What is a `.dockerignore` file?](#what-is-a-dockerignore-file)**
	- **[Benefits of using a `.dockerignore` file](#benefits-of-using-a-dockerignore-file)**
		- **[Cache invalidation](#cache-invalidation)**
		- **[Reduces the image size](#reduces-the-image-size)**
		- **[Security Issues](#security-issues)**
	- **[How Does`.dockerignorefile` works?](#how-does-dockerignorefile-works)**
		- **[Example of `.dockerignore` file](#example-of-dockerignore-file)**

## What is a Dockerfile?

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

##### List of the instructions

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
-  **Note** : By design, the `VOLUME` directive in the Dockerfile only defines that a volume should be created for a specific path in the image (/container when running).The name of the volume should never be dictated by an image, because that's something that should be defined at runtime;

```dockerfile
ONBUILD RUN composer install --optimize-autoloader --no-dev
```

- `ONBUILD`executes after the current Dockerfile build completes. 
- The `ONBUILD` instruction may not trigger `FROM`, `MAINTAINER`, or `ONBUILD` instructions.

```dockerfile
HEALTHCHECK <options> CMD <command>

HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost/ || exit 1

#Disable any healthcheck inherited from the base image
HEALTHCHECK NONE
```

- `HEALTHCHECK`check container health by running a command inside the container.(Instructs Docker how to test a container to check that it is still working)
- Whenever a health check passes, it becomes `healthy`. After a certain number of consecutive failures, it becomes `unhealthy` You can use `docker ps`command to check the status.
-  The `<options>` that can appear are...
    - `--interval=<duration>` (default: `30s`) In practice - should be relatively long (`10m`)
    - `--timeout=<duration>` (default: `30s`)
    - `--retries=<number>` (default: `3`)
    - `--start_period=<duration>` (default: `0s`)
    -  `--start-interval=DURATION` (default: `5s`)
- The health check will first run `interval` seconds after the container is started, and then again `interval` seconds after each previous check completes. If a single run of the check takes longer than `timeout` seconds then the check is considered to have failed. It takes `retries` consecutive failures of the health check for the container to be considered `unhealthy`.
- `start period` provides initialization time for containers that need time to bootstrap. Probe failure during that period will not be counted towards the maximum number of retries. However, if a health check succeeds during the start period, the container is considered started and all consecutive failures will be counted towards the maximum number of retries.
- `start interval` is the time between health checks during the start period.
- There can only be one `HEALTHCHECK` instruction in a Dockerfile. If you list more than one then only the last `HEALTHCHECK` will take effect.
- The command's exit status indicates the health status of the container.
    - `0`: **success** - the container is healthy and ready for use
    - `1`: **unhealthy** - the container is not working correctly
    - `2`:  **reserved** - do not use this exit code

- The first 4096 bytes of stdout and stderr from the `<command>` are stored and can be queried with `docker inspect`.
- When the health status of a container changes, a `health_status` event is generated with the new status.



## `.dockerignore` file

##### What is a `.dockerignore` file?

A `.dockerignore` is a configuration file that describes files and directories that you want to exclude when building a Docker image.

Usually, you put the Dockerfile in the root directory of your project, but there may be many files in the root directory that are not related to the Docker image or that you do not want to include. `.dockerignore` is used to specify such unwanted files and not include them in the Docker image.

##### Benefits of using a `.dockerignore` file

The `.dockerignore` file is helpful to avoid inadvertently sending files or directories that are large or contain sensitive files to the daemon or avoid adding them to the image using the `ADD` or `COPY` commands. Those are some benefits of using such file : 

###### Cache invalidation

If you have frequently updated files (git history, test results, etc.) in your working directory, the cache will be regenerated every time you run Docker build. Therefore, if you include the directory with such files in the context, each build will take a lot of time. Consequently, you can prevent the cache from generating at build time by specifying `.dockerignore`.

###### Reduces the image size

By specifying files not to be included in the context in `.dockerignore`, the size of the image can be reduced. Reducing the size of the Docker image has the following advantages These benefits are important because the more instances of a service you have, such as microservices, the more opportunities you have to exchange Docker images.

- Faster speed when doing `Docker pull/push`.
- Faster speed when building Docker images.

**Examples**: it is recommended to add directories such as `node_modules`, `vendor`... to `.dockerignore` because of their large file size.

###### Security Issues

If a Docker image file contains sensitive information such as credentials, it becomes a security issue. For example, uploading a Docker image containing files with credential information such as `.aws` or `.env` to a Docker repository such as the public DockerHub will cause those credential information to be compromised.

Don't forget to exclude the `.git` directory at this point. If you have committed sensitive information in the past but have not erased it, it can cause serious problems. Git history is not required to be included in Docker images, so be sure to include it in your `.dockerignore` file.

##### How does `.dockerignorefile` works?

Before sending the Docker build context (Dockerfile, files you want to send inside the Docker image, and other things needed when building the Docker image) to the Docker daemon, in the root directory of the build context look for a file named `.dockerignore` in the root directory of the build context. If this file exists, the CLI will exclude any files or directories from the context that match the pattern written in the file. Therefore, the files and directories described in the `.dockerignore` file will not be included in the final built Docker image.

###### Example of `.dockerignore` file

```bash
**/node_modules/
npm-debug.log
.env
.aws
.editorconfig
.git
.gitignore
LICENSE
README.md
src/vendor
**/*.swp
Dockerfile
docker-compose*
```