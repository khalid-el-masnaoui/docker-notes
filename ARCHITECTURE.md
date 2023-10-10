
## Docker Architecture

Notes about docker , docker architecture and how it works and other docker concepts.

## What is Docker

Docker is a software framework for building, running, and managing containers on servers and the cloud. The term "docker" may refer to either the tools (the commands and a daemon) or to the `Dockerfile` file format.

It used to be that when you wanted to run a web application, you bought a server, installed _Linux_, set up a LEMP stack, and ran the app. If your app got popular, you practiced good _load balancing_ by setting up a second server to ensure the application wouldn't crash from too much traffic.

Times have changed, though, and instead of focusing on single servers, the Internet is built upon arrays of inter-dependent and redundant servers in a system commonly called "the cloud". Thanks to innovations like _Linux kernel namespaces and cgroups_, the concept of a server could be removed from the constraints of hardware and instead became, essentially, a piece of software. These software-based servers are called [containers](#what-are-docker-containers),  and they're a hybrid mix of the Linux OS they're running on plus a hyper-localized runtime environment (the contents of the container).

## What are Docker Containers

Container technology can be thought of as three different categories:

- **_Builder_**:  a tool or series of tools used to build a container, such as a Dockerfile for Docker.
- **_Engine_**: an application used to run a container. For Docker, this refers to the _docker_ command and the `dockerd` daemon.
- **_Orchestration_**: technology used to manage many containers, including Kubernetes and OKD.

_Containers_ often deliver both an application and configuration, meaning that you don't have to spend as much time getting an application in a container to run compared to when an application is installed from a traditional source. [Dockerhub](http://hub.docker.com/) are repositories offering [images](#What-are-docker-images) for use by container engines.

The greatest appeal of containers, though, is their ability to "die" gracefully and re-spawn when load balancing demands it. Whether a container's demise is caused by a crash or because it's simply no longer needed because server traffic is low, containers are "cheap" to start, and they're designed to seamlessly appear and disappear. Because containers are meant to be ephemeral and to spawn new instances as often as required, it's expected that monitoring and managing them is not done by a human in real time, but is instead automated.

## What are Docker Images

##### Docker image

A Docker image is a file used to execute code in a Docker container. Docker images act as a set of instructions to build a Docker [container](#what-are-docker-containers), like a template. Docker images also act as the starting point when using Docker. An image is comparable to a _snapshot in virtual machine (VM) environments_.

Docker is used to create, run and deploy applications in containers. A Docker image contains application code, libraries, tools, dependencies and other files needed to make an application run. When a user runs an image, it can become _one or many instances of a container_.

Docker images have _multiple layers_, each one originates from the previous layer but is different from it. The layers speed up Docker builds] while increasing re-usability and decreasing disk use. Image layers are also **read-only** files. Once a container is created, a **writable layer** is added on top of the unchangeable images, allowing a user to make changes.

A Docker image has everything needed to run a containerized application, including code, config files, environment variables, libraries and run-times. When the image is deployed to a Docker environment, it can be executed as a Docker container. The `docker run command` creates a container from a specific image.


##### Docker image repositories

Docker images are a reusable asset -- deployable on any host. Developers can take the static image layers from one project and use them in another. This saves the user time, because they do not have to recreate an image from scratch.

Docker images get stored in private or public repositories, such as those in the [Docker Hub](http://hub.docker.com) cloud registry service, from which users can deploy containers and test and share images.

_Official images_ are ones Docker produces, while _community images_ are images Docker users create.

Users can also create new images from existing ones and use the `docker push` command to upload custom images to the Docker Hub.

**Note** :Docker images can be created through [Dockerfiles](dockerfile.md)

![](images/docker_container_images.jpg)
