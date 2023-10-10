
## Docker Architecture

Notes about docker , docker architecture and how it works and other docker concepts.

## What is Docker

Docker is a software framework for building, running, and managing containers on servers and the cloud. The term "docker" may refer to either the tools (the commands and a daemon) or to the `Dockerfile` file format.

It used to be that when you wanted to run a web application, you bought a server, installed _Linux_, set up a LEMP stack, and ran the app. If your app got popular, you practiced good _load balancing_ by setting up a second server to ensure the application wouldn't crash from too much traffic.

Times have changed, though, and instead of focusing on single servers, the Internet is built upon arrays of inter-dependent and redundant servers in a system commonly called "the cloud". Thanks to innovations like _Linux kernel namespaces and cgroups_, the concept of a server could be removed from the constraints of hardware and instead became, essentially, a piece of software. These software-based servers are called [containers](#what-are-docker-container),  and they're a hybrid mix of the Linux OS they're running on plus a hyper-localized runtime environment (the contents of the container).

## What are Docker Containers

Container technology can be thought of as three different categories:

- **_Builder_**:  a tool or series of tools used to build a container, such as a Dockerfile for Docker.
- **_Engine_**: an application used to run a container. For Docker, this refers to the _docker_ command and the `dockerd` daemon.
- **_Orchestration_**: technology used to manage many containers, including Kubernetes and OKD.
    

Containers often deliver both an application and configuration, meaning that a sysadmin doesn't have to spend as much time getting an application in a container to run compared to when an application is installed from a traditional source. [Dockerhub](http://hub.docker.com/) and [Quay.io](http://quay.io/) are repositories offering images for use by container engines.

The greatest appeal of containers, though, is their ability to "die" gracefully and respawn when load balancing demands it. Whether a container's demise is caused by a crash or because it's simply no longer needed because server traffic is low, containers are "cheap" to start, and they're designed to seamlessly appear and disappear. Because containers are meant to be ephemeral and to spawn new instances as often as required, it's expected that monitoring and managing them is not done by a human in real time, but is instead automated.