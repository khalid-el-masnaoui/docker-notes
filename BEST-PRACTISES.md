
## Docker Best Practices

Notes about some of the best docker practices for improving Docker security, optimize the image size, writing cleaner and more maintainable Dockerfiles, handling volumes permissions and many more.

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

##### Docker container security best practices

The settings you apply to your Docker containers at runtime affect the security of your containerized applications, as well as your Docker host. Here are some techniques which help prevent containers from posing a threat.