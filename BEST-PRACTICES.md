
## Docker Best Practices

Notes about some of the best docker practices for improving Docker security, optimize the image size, writing cleaner and more maintainable _Dockerfiles_, handling volumes permissions and many more.


# Table Of Contents

- **[Docker Security Best Practices](#docker-security-best-practices)**
	- **[Docker daemon security](#docker-daemon-security)**
		- **[Don't expose the Docker daemon socket](#dont-expose-the-docker-daemon-socket)**
		- **[Use TLS if you must expose the daemon socket](#docker-daemon-security)**
		- **[Enable rootless mode where possible](#use-tls-if-you-must-expose-the-daemon-socket)**
		- **[Keep Docker updated](#keep-docker-updated)**
		- **[Disable inter-container communication](#disable-inter-container-communication)**
		- **[Enable OS-level security protections (SELinux/Seccomp/AppArmor)](#enable-os-level-security-protections-selinuxseccompapparmor)**
		- **[Harden your host](#harden-your-host)**
		- **[Enable user namespace remapping](#enable-user-namespace-remapping)**
	- **[Docker image security](#docker-image-security)**
		- **[Use trusted/minimal base images](#use-trustedminimal-base-images)**
		- **[Regularly rebuild your images](#regularly-rebuild-your-images)**
		- **[Use image vulnerability scanners](#use-image-vulnerability-scanners)**
		- **[Use Docker content trust to verify image authenticity](#use-docker-content-trust-to-verify-image-authenticity)**
		- **[Lint your Dockerfiles to detect unsafe misconfigurations](#lint-your-dockerfiles-to-detect-unsafe-misconfigurations)**
	- **[Docker container security](#docker-container-security)**
		- **[Don't expose unnecessary ports](#dont-expose-unnecessary-ports)**
		- **[Don't start containers in privileged mode](#dont-start-containers-in-privileged-mode)**
		- **[Drop capabilities when you start containers](#drop-capabilities-when-you-start-containers)**
		- **[Limit the resources access for containers](#limit-the-resources-access-for-containers)**
		- **[Ensure container processes run as a non-root user](#ensure-container-processes-run-as-a-non-root-user)**
		- **[Prevent containers from escalating privileges](#prevent-containers-from-escalating-privileges)**
		- **[Use read-only filesystem mode](#use-read-only-filesystem-mode)**
- **[Docker volumes and files permissions Best Practices](#docker-volumes-and-files-permissions-best-practices)**
	- **[Introduction](#introduction)**
	- **[Solutions](#solutions)**
- **[Docker volumes and files permissions Best Practices](#other-docker-best-practices)**
	- **[Layer sanity](#layer-sanity)**
	- **[Include health-checks](#include-health-checks)**

## Docker Security Best Practices

##### Docker daemon security

Docker’s architecture is daemon-based: the client CLI you interact with communicates with a separate background service to carry out actions such as building images and starting containers. It’s critical to protect the daemon because anyone with access to it can execute Docker commands on your host.

######  Don't expose the Docker daemon socket

The Docker daemon is normally exposed via a Unix socket at `/var/run/docker.sock`. It’s possible to optionally configure the daemon to listen on a _TCP socket_ too, which allows remote connections to the Docker host from another machine.

This should be avoided because it presents an additional attack vector. Accidentally exposing the TCP socket on your public network would allow anyone to send commands to the Docker API, without first requiring physical access to your host. Keep TCP disabled unless your use case demands remote access.

Do not expose `/var/run/docker.sock` to other containers. If you are running your docker image with `-v /var/run/docker.sock://var/run/docker.sock` or similar, you should change it. Remember that mounting the socket read-only is not a solution but only makes it harder to exploit.

###### Use TLS if you must expose the daemon socket

When there’s no alternative to using TCP, it’s essential to [protect the socket with TLS](https://docs.docker.com/engine/security/protect-access). This will ensure access is only granted to clients that present the correct certificate key.

TCP with TLS is still a potential risk because any client with the certificate can interact with Docker. It’s also possible to use an SSH-based connection to communicate with the Docker daemon, which allows you to reuse your existing SSH keys.


###### Enable rootless mode where possible

Docker defaults to running both the daemon and your containers as `root`. This means that vulnerabilities in the daemon or a container could allow attackers to breakout and run arbitrary commands on your host.

[Rootless mode](https://docs.docker.com/engine/security/rootless) is an optional feature that allows you to start the Docker daemon without using root. It’s more complex to set up and has some [limitations](https://docs.docker.com/engine/security/rootless/#known-limitations), but it provides a useful extra layer of protection for security-sensitive production environments.

###### Keep Docker updated

One of the easiest ways to maintain Docker security is to stay updated with new releases. Docker regularly issues patches that fix newly found security problems. Running an old version makes it more likely that you’re missing protections for exploitable vulnerabilities.

Regularly apply any updates offered by your OS package manager to ensure you’re protected. 

###### Disable inter-container communication

Docker normally allows arbitrary communication between the containers running on your host. Each new container is automatically added to the `docker0` bridge network, which allows it to discover and contact its peers.

Keeping inter-container communication (ICC) enabled is risky because it could permit a malicious process to launch an attack against neighboring containers. You should increase your security by launching the Docker daemon with ICC disabled (using the `--icc=false` flag), then permit communications between specific containers by manually creating networks.

###### Enable OS-level security protections (SELinux/Seccomp/AppArmor)

Ensuring OS-level security systems are active helps defend against malicious activity originating inside containers and the Docker daemon. Docker supports policies for [SELinux](https://github.com/containers/container-selinux), [Seccomp](https://docs.docker.com/engine/security/seccomp), and [AppArmor](https://docs.docker.com/engine/security/apparmor); keeping them enabled ensures sane defaults are applied to your containers, including restrictions for [dangerous system calls](https://docs.docker.com/engine/security/seccomp/#significant-syscalls-blocked-by-the-default-profile).

The default `seccomp` profile provides a sane default for running containers with seccomp and disables around 44 system calls out of 300+. It is moderately protective while providing wide application compatibility.

###### Harden your host

Docker security is only as good as the protection surrounding your host. You should fully harden your host environment by taking steps such as regularly updating your host’s OS and kernel, enabling firewalls and network isolation, and restricting direct host access to just the administrators who require it.

Neglecting basic security measures will undermine the strongest container protections.

###### Enable user namespace remapping

[User namespace remapping](https://docs.docker.com/engine/security/userns-remap) is a Docker feature that converts host UIDs to a different unprivileged range inside your containers. This helps to prevent privilege escalation attacks, where a process running in a container gains the same privileges as its UID has on your host.

User namespace remapping assigns the container a range of UIDs from 0 to 65536 that translate to unprivileged host users in a much higher range. To enable the feature, you must start the Docker daemon with a `--userns-remap` flag that specifies how the remapping should occur.


##### Docker image security

Once you’ve tightened the security around your Docker daemon installation, it’s important to also review the images that you use. A compromised image can harbor security threats that form the basis of a successful attack.

###### Use trusted/minimal base images

Only select trusted base images for the `FROM` instructions in your [Dockerfiles](#DOCKERFILE.md). You can easily find these images by filtering using the “Docker Official Image” and “Verified Publisher” options [on Docker Hub](https://hub.docker.com/search?q=&image_filter=official%2Cstore). An image that’s published by an unknown author or which has few downloads might not contain the content you expect.

It’s also advisable to use minimal images (such as [Alpine-based variants](https://hub.docker.com/_/alpine)) where possible. These will have smaller download sizes and should contain fewer OS packages, which reduces your attack surface.

###### Regularly rebuild your images

Regularly rebuild your images from your Dockerfiles to ensure they include updated OS packages and dependencies. Built images are immutable, so package bug fixes and security patches released after your build won’t reach your running containers.

Periodically rebuilding your images and restarting your containers is the best way to prevent stale dependencies being used in production. You can automate the container replacement process by using a tool such as [Watchtower](https://containrrr.dev/watchtower).

###### Use image vulnerability scanners

Scanning your built images for vulnerabilities is one of the most effective ways to inform yourself of problems. Scan tools are capable of identifying which packages you’re using, whether they contain any vulnerabilities, and how you can address the problem by upgrading or removing the package.

You can perform a scan by running the [docker scout](https://docs.docker.com/scout) command (previously docker scan) or an external tool such as [Anchore](https://anchore.com/) or [Trivy](https://github.com/aquasecurity/trivy). Scanning each image you build will reveal issues before the image is used by containers running in production. It’s a good idea to include these scans as jobs in your CI pipeline.

###### Use Docker content trust to verify image authenticity

Before starting a container, you need to be sure that the image you’re using is authentic. An attacker could have uploaded a malicious replacement to your registry or intercepted the download to your host.

Docker [Content Trust](https://docs.docker.com/engine/security/trust) is a mechanism for signing and verifying images. Image creators can sign their images to prove that they authored them; consumers who pull images can then verify the trust by comparing the image’s public signature.

Docker [can be configured](https://docs.docker.com/engine/security/trust/#client-enforcement-with-docker-content-trust) to prevent the use of unsigned or unverifiable images. This provides a safeguard against potentially tampered content.

###### Lint your Dockerfiles to detect unsafe misconfigurations

Linting your Dockerfiles before you build them is an easy way to spot common mistakes  that could pose a security risk. Linters such as [Hadolint](https://github.com/hadolint/hadolint) check your Dockerfile instructions and flag any issues that contravene best practices.

Fixing detected problems before you build will help ensure your images are secure and reliable. This is another process that’s worth incorporating into your CI pipelines.

##### Docker container security

The settings you apply to your Docker containers at runtime affect the security of your containerized applications, as well as your Docker host. Here are some techniques which help prevent containers from posing a threat.

###### Don't expose unnecessary ports

Exposing container ports unnecessarily (using the `-p` or `--port` flag for `docker run`) can increase your attack surface by allowing external processes to probe inside the container. Only ports which are actually needed by the containerized application (typically those listed as Dockerfile `EXPOSE` instructions) should be opened.

###### Don't start containers in privileged mode

Using [privileged mode](https://docs.docker.com/engine/reference/commandline/run/#privileged) (`--privileged`) is a security risk that should be avoided unless you’re certain it’s required. Containers that run in privileged mode are granted _all_ available Linux capabilities and have some cgroups restrictions lifted. This allows them to achieve almost anything that the host machine can.

Containerized apps very rarely require privileged mode. It’s typically only useful when you’re running an application that needs full access to your host or the ability to manage other Docker containers.

###### Drop capabilities when you start containers

Even the default set of [Linux capabilities](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities) granted by Docker can be too permissive for production use. They include the ability to change file UIDs and GIDs, kill processes, and bypass file read, write, and execute permission checks.

It’s good practice to drop capabilities that your container doesn’t need. The `docker run` command’s `--cap-drop` and `--cap-add` flags allow you to remove and grant them. The following example drops every capability, then adds back `CHOWN` to permit file ownership changes:

```bash
$ docker run --cap-drop=ALL --cap-add=CHOWN example-image:latest
```

###### Limit the resources access for containers

Docker doesn’t automatically apply any resource constraints to your containers. Containerized processes are free to use unlimited CPU and memory, which could impact other applications on your host. Setting limits for these resources helps to defend against denial-of-service (DoS) attacks.

Limit a container’s memory allowance by including the `-m` or `--memory` flag with your `docker run` commands:

```bash
$ docker run -m=128m example-image:latest
```

To set a CPU allowance, supply the `--cpus` flag. You must specify the number of CPU cores you want to make available:

```bash
$ docker run --cpus=2 example-image:latest
```


###### Ensure container processes run as a non-root user

Containers default to running as `root` but this can be changed by either including a `USER` instruction in your Dockerfile or setting the `docker run` command’s `--user` flag:

```bash
$ docker run --user=1000 example-image:latest
```

Either of these methods ensure your container will execute as a specific non-root user. This minimizes the risk of container breakout attacks by preventing the containerized process from running commands as `root` on your host.

###### Prevent containers from escalating privileges

Containers can usually escalate their privileges by calling the `setuid` and `setgid` binaries. This is a security risk because the containerized process could use `setuid` to effectively become `root`.

To prevent this, you should set the `no-new-privileges` security option when you start your containers:

```bash
$ docker run –security-opt=no-new-privileges:true example-image:latest
```

When this flag is used, calls to `setuid` and `setgid` will have no effect. This prevents the container from acquiring new privileges.

###### Use read-only filesystem mode

Few containerized applications need to write directly to their filesystem. Opting them into Docker’s read-only mode prevents filesystem modifications, except to designated volume mount locations. This will block an intruder from making malicious changes to the content within the container, such as by replacing binaries or configuration files.

```bash
$ docker run --read-only example-image:latest
```

## Docker volumes and files permissions Best Practices

##### Introduction

The whole issue with file permissions in docker containers comes from the fact that the Docker host shares file permissions with containers (at least, in Linux). Let me remind you here that file permissions on _bind mounts_ are shared between the host and the containers. Whenever we create a file on host using a user with `UID x`, this files will have x as owner UID inside the container, too. And this will happen no matter if there is a user with UID equal to x inside the container. The same holds whenever a file is created inside the container by a user with UID equal to y (or a process running under this user). This file will appear on the host with owner UID equal to y.

Docker containers share the same kernel with the host (the server where Docker daemon runs). Since there is only one kernel, there is also only one set of UIDs and GIDs for the host and all the containers. I am saying “UIDs and GIDs” instead of “users and groups” because the second one is a more generic terminology and I want to point out the difference. User names and group names are not shared! They are not part of the kernel. They are managed by external (to the kernel) tools (like /etc/passwd) who map user names to UID and group names to GIDs. So, you can have the same UID mapped to different user name in each container or the host. And because of that the root user on host is the same with the root user in any container (they have the same UID).

<p align="center">
<img src="./images/docker_kernel_uid.png"/>
</p>

- Security issues may rise in a production system. If a container is compromised and the container is executed as root (uid = 0), then the intruder has access to any file of the host filesystem that has been loaded to the container filesystem through a mount. The owner UID of files that belong to the host root will be 0 in the container. So, they will be accessible to the intruder.

- Another issue is related to the user under which the build process of a docker image is executed. This user is the user under which `RUN`, `CMD` and `ENTRYPOINT` directives of Dockerfile are executed. So, it determines the permissions of files and directories that are created during the build process (e.g when we use composer or Node to retrieve files from external repositories into the container). If this user is the “root”, then these files will not be accessible from web server or the CGI server, except if the server is running as root (something rare and undesirable). What is more, if we use bind mounts for development, these files will probably be not accessible from our IDE (which runs on the host under our local user).

- Files created by the developer (created on host and mounted to the container) will have as owner uid the one of the developer’s host account and will probably be not accessible by the web server that runs inside the container. And this is true not only for the image build process but also for changes that happen in run-time.

##### Solutions

_Reminder_ : Most files are deployed to a container through:

- (a) a `COPY` directive in dockerfile , (during the image build process)
- (b) through a `docker cp` command, (usually after a docker create command that creates but doesn’t start yet the container)
- (c) mounting of a host directory (e.g a bind mount defined in `docker run` command or in the `docker-compose.yml`),
- (d) a entry script that is executed during the image build process.

In cases (a) and (d), the onwer of the files in the container will be the user under which the container is running. In cases (b) and (c), the file permissions are “inhereted” from host’s filesystem.

###### 1.Create a non-root user in the container and run the container under such user

Some services (like Nginx, MySql or PHP-FPM...) create a user during their installation, so we don’t have to do it ourselves.

```dockerfile
#add user and group
RUN groupadd -f www-data && \
    (id -u www-data &> /dev/null || useradd -G www-data www-data -D)

# run the container under such user
USER www-data
```

###### 2.Remapping the UIDs/GIDs of the container's user to the UIDs/GIDs of the host's user

```dockerfile
#assign the created user same UID AND GUID OF the host for the mounted dir owner
ARG UID
ARG GID

RUN usermod -u $UID www-data
RUN groupmod -g $GID www-data
```

then you can assign the UID and GID during the build either by specifying
`--build-arg="UID=$(id -u)" --build-arg="GID=$(id -g)"` in the `docker build` command or using `docker-compose.yml`as below: 

```yml
version: '3.8'
# Services
services:
  #Nginx Service
  nginx:
    build:
      context: .
      dockerfile: docker/nginx.Dockerfile
      args:
        UID: ${XUID}
        GID: ${XGID}
```

and then 

```bash
# To your ~/.bashrc file append these two lines:
$ export XUID=$(id -u) 
$ export XGID=$(id -g)

#Then refresh the file (or open a new terminal):
$ source ~/.bashrc # or '. ~/.bashrc'
```

###### 3.Adjusting the permission during the build

_Examples_ : 
```dockerfile
# Copied files/directorties
COPY --chown=www-data:www-data src/ /var/www/html/
RUN chown -r app:app /var/www/html/ #(with above command no need to tun this)

# PID directory
RUN install -d -m 0755 -o www-data -g www-data /run/php-fpm
RUN install -o www-data -g www-data /dev/null /var/run/nginx.pid

# Logs
RUN install -o mysql -g mysql -d /var/log/mysql && \
    install -o mysql -g mysql /dev/null /var/log/mysql/error.log && \
```

**Note** : You can also enable userns-remap on Docker daemon

## Other Docker Best practices

##### Layer sanity

Remember that order in the Dockerfile instructions is very important.

Since `RUN`, `COPY`, `ADD`, and other instructions will create a new container layer, grouping multiple commands together will reduce the number of layers.

For example, instead of:

```dockerfile
FROM ubuntu 
RUN apt-get install -y wget 
RUN wget https://…/downloadedfile.tar 
RUN tar xvzf downloadedfile.tar 
RUN rm downloadedfile.tar 
RUN apt-get remove wget
```

It would be a Dockerfile best practice to do:

```dockerfile
FROM ubuntu 
RUN apt-get install wget && wget https://…/downloadedfile.tar && tar xvzf downloadedfile.tar && rm downloadedfile.tar && apt-get remove wget
```
Also, place the commands that are less likely to change, and easier to cache, first.

Instead of:

```dockerfile 
FROM ubuntu 
COPY source/* . 
RUN apt-get install nodejs 
ENTRYPOINT ["/usr/bin/node", "/main.js"]
```

It would be better to do:

```dockerfile
FROM ubuntu 
RUN apt-get install nodejs 
COPY source/* . 
ENTRYPOINT ["/usr/bin/node", "/main.js"]
```

  
The `nodejs` package is less likely to change than our application source.

Please remember that executing a `rm` command removes the file on the next layer, but it is still available and can be accessed, as the final image filesystem is composed from all the previous layers.

So **don’t copy confidential files and then remove them**, they will be not visible in the final container filesystem but still be easily accessible.

##### Include health-checks

Adding a `HEALTHCHECK` to your Docker image is important in ensuring your application runs correctly and ensuring the container reliability . A `HEALTHCHECK` is a command that Docker runs to check the status of your application. If the command returns a non-zero exit code, Docker will consider the container unhealthy.

Adding a `HEALTHCHECK` to your Docker image is a simple but effective way to ensure your application runs correctly and detects any issues before they become critical.

Read more about this [here](DOCKERFILE.md)