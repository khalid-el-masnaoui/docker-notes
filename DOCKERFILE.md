
## Dockerfile Notes

Notes about _Dockerfile_, a complete list of the instructions used and some best practices.


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


## Dockerfile reference