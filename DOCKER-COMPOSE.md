
## Docker-Compose Notes

Notes about _docker-compose_ file, a complete docker-compose file reference, the main commands  and some best practices.


## What is a docker-compose file?

###### Introduction

**Docker Compose** is a Docker tool used to define and run multi-container applications. With Compose, you use a `YAML` file to configure your application’s services and create all the app’s services from that configuration.In other words,  `docker-compose` is an **automated multi-container workflow.**

the most popular features of Docker Compose are:

- Multiple isolated environments on a single host
- Preserve volume data when containers are created
- Only recreate containers that have changed
- Variables and moving a composition between environments
- Orchestrate multiple containers that work together

###### Dockerfile vs Docker Compose

Both Dockerfile and Docker Compose are tools in the Docker image ecosystem. **Dockerfile** is a text file that contains an image, and the commands a developer can call to assemble the image. The commands are typically simple processes like installing dependencies, copying files, and configuring settings.

**Docker Compose** is a tool for defining and running multi-container Docker applications. Information describing the services and networks for an application are contained within a `YAML` file, called `docker-compose.yml`.

One of the base functions of Docker Compose is to `build` images from _Dockerfiles_. However, Docker Compose is capable of **orchestrating** the containerization and deployment of multiple software packages. You can select which images are used for certain services, set environment-specific variables, configure network connections, and much more.

Below are the three necessary steps that begin most Docker workflows:

- Building a `Dockerfile` for each image you wish to add
- Use `Docker Compose` to assemble the images with the `build` command.
- Specify the path to individual Dockerfiles using the `build` command in conjunction with `/path/to/dockerfiles`

In summary, _Dockerfiles_ define the instructions to a single image in an application, but _Docker Compose_ is the tool that allows you to create and manipulate a multi-container application.

<p align="center">
<img src="./images/docker_compose.png"/>
</p>

## Docker Compose file structure

Docker Compose files work by applying multiple commands that are declared within a single `docker-compose.yml` configuration file.

There are four main things almost every Compose-File should have which include:

- The version of the compose file
- The services which will be built
- All used volumes
- The networks which connect the different services

The basic structure of a Docker Compose YAML file looks like this:

```yaml
version: '3.8'

services:
   db:
     image: mysql:8.0.34
     volumes:
       - db_data:/var/lib/mysql
     restart: on-failure
     environment:
       MYSQL_ROOT_PASSWORD: somewordpress
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress
	 networks:
	   - malikdha-network	 
    php:
        build:
            context: .
            dockerfile: docker/php.Dockerfile
        ports:
            - '9000:9000'
        env_file:
            - '.env'
        volumes:
            - type: bind
              source: src/
              target: /var/www/html
        networks:
            - malidkha-network
        depends_on:
            db:
volumes:
    db_data:
networks:
	malikdha-network:
```

As you can see this file contains a basic PHP application including the MySQL database. Each of these services is treated as a separate container that can be swapped in and out when you need it.

The db service is built from an image being pulled from DockerHub , while the php service is built from a custom dockerfile 

## Docker Compose commands

##### Basic commands 

We will list and discuss the main and most commonly used `docker-compose`commands.

```bash
# command builds or rebuild images in the docker-compose.yml file.
$ docker-compose build

# This is similar to the docker run command. It will create containers from images built for the services mentioned in the compose file.
$ docker-compose run

# This command does the work of the docker-compose build and docker-compose run commands. It builds the images if they are not located locally and starts the containers. If images are already built, it will fork the container directly.
# Builds, (re)creates, starts, and attaches to containers for a service.
$ docker-compose up

# Stops containers and removes containers, networks, volumes, and images created by up.
$ docker-compose down

# Forces running containers to stop by sending a `SIGKILL` signal
$ docker-compose kill 

# Starts existing containers for a service.
$ docker-compose start

# Restarts all stopped and running services, or the specified services only.
$ docker-compose restart

# Stops running containers without removing them.
$ docker-compose stop

# Pauses running containers of a service.
$ docker-compose pause

# Unpauses paused containers of a service.
$ docker-compose unpause

# Removes stopped service containers.
$ docker-compose rm

# List images used by the created containers
$ docker-compose images 

# Lists containers.
$ docker-compose ps


# Displays log output from services.
$ docker-compose logs
```

**Options & Flags** 

`docker-compose up`:
- `--build`: Build images before starting containers.
- `--detach`or `-d`: Detached mode: Run containers in the background
- you find [here the complete list of options](https://docs.docker.com/engine/reference/commandline/compose_up/#options) for this command.