
## Docker-Compose Notes

Notes about _docker-compose_ file, a complete docker-compose file commands reference and some best practices.


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
