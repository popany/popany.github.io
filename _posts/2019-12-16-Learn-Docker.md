---
layout: post
title: Learn - Docker as Build Environment
---

## Dockerfile

[How to Build Docker Images with Dockerfile](https://linuxize.com/post/how-to-build-docker-images-with-dockerfile/)

[Dockerfile reference](https://docs.docker.com/engine/reference/builder/)

## Install

[How To Install and Use Docker on Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-18-04)

[Setting Up Docker for Windows and WSL to Work Flawlessly](https://nickjanetakis.com/blog/setting-up-docker-for-windows-and-wsl-to-work-flawlessly)

[Arch Linux Wiki - Docker](https://wiki.archlinux.org/index.php/Docker)

## Use

[Docker as Build Environment](https://www.rainerhahnekamp.com/en/docker-build-environment/)

[Use Docker images as build environments](https://confluence.atlassian.com/bitbucket/use-docker-images-as-build-environments-792298897.html)

[How To Build a Local Development Environment Using Docker](https://masterzendframework.com/docker-development-environment/)

[Using Docker for Reproducible Build Environments](https://sweetcode.io/using-docker-reproducible-build-environments/)

[Docker C++ development and CI](https://stackoverflow.com/questions/42748040/docker-c-development-and-ci)

## [Best practices for writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

Docker builds images automatically by reading the instructions from a Dockerfile -- a text file that contains all commands, in order, needed to build a given image. 

A Docker image consists of read-only layers each of which represents a Dockerfile instruction. The layers are stacked and each one is a delta of the changes from the previous layer.

When you run an image and generate a container, you add a new writable layer (the “container layer”) on top of the underlying layers. All changes made to the running container, such as writing new files, modifying existing files, and deleting files, are written to this thin writable container layer.

### General guidelines and recommendations

#### Create ephemeral containers

The image defined by your `Dockerfile` should generate containers that are as ephemeral as possible. By “ephemeral”, we mean that the container can be stopped and destroyed, then rebuilt and replaced with an absolute minimum set up and configuration.

#### Understand build context

When you issue a docker build command, the current working directory is called the build context. By default, the `Dockerfile` is assumed to be located here, but you can specify a different location with the file flag (`-f`). Regardless of where the `Dockerfile` actually lives, all recursive contents of files and directories in the current directory are sent to the Docker daemon as the build context.

Inadvertently including files that are not necessary for building an image results in a larger build context and larger image size. This can increase the time to build the image, time to pull and push it, and the container runtime size. To see how big your build context is, look for a message like this when building your `Dockerfile`:

    Sending build context to Docker daemon  187.8MB

#### Pipe Dockerfile through `stdin`
