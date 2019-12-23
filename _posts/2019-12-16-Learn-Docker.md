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

## See

[Should You Try to Set Environment Variables Based on a Command Output While Building a Docker Image?](https://vsupalov.com/set-dynamic-environment-variable-during-docker-image-build/)

[6 Docker Basics You Should Completely Grasp When Getting Started](https://vsupalov.com/6-docker-basics/)

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

Docker has the ability to build images by piping `Dockerfile` through `stdin` with a local or remote build context. Piping a `Dockerfile` through `stdin` can be useful to perform one-off builds without writing a `Dockerfile` to disk, or in situations where the `Dockerfile` is generated, and should not persist afterwards.

For example, the following commands are equivalent:

    echo -e 'FROM busybox\nRUN echo "hello world"' | docker build -

    docker build -<<EOF
    FROM busybox
    RUN echo "hello world"
    EOF

##### Build an image using a `Dockerfile` from `stdin`, without sending build context

Use this syntax to build an image using a `Dockerfile` from `stdin`, without sending additional files as build context. The hyphen (`-`) takes the position of the `PATH`, and instructs Docker to read the build context (which only contains a Dockerfile) from `stdin` instead of a directory:

    docker build [OPTIONS] -

The following example builds an image using a `Dockerfile` that is passed through `stdin`. No files are sent as build context to the daemon.

    docker build -t myimage:latest -<<EOF
    FROM busybox
    RUN echo "hello world"
    EOF

Omitting the build context can be useful in situations where your `Dockerfile` does not require files to be copied into the image, and improves the build-speed, as no files are sent to the daemon.
If you want to improve the build-speed by excluding some files from the build- context, refer to exclude with `.dockerignore`.

##### Build from a local build context, using a `Dockerfile` from `stdin`

Use this syntax to build an image using files on your local filesystem, but using a `Dockerfile` from `stdin`. The syntax uses the `-f` (or `--file`) option to specify the `Dockerfile` to use, using a hyphen (`-`) as filename to instruct Docker to read the `Dockerfile` from `stdin`:

    docker build [OPTIONS] -f- PATH

The example below uses the current directory (`.`) as the build context, and builds an image using a `Dockerfile` that is passed through `stdin` using a here document.

    # create a directory to work in
    mkdir example
    cd example

    # create an example file
    touch somefile.txt

    # build an image using the current directory as context, and a Dockerfile passed through stdin
    docker build -t myimage:latest -f- . <<EOF
    FROM busybox
    COPY somefile.txt .
    RUN cat /somefile.txt
    EOF

##### Build from a remote build context, using a `Dockerfile` from `stdin`

Use this syntax to build an image using files from a remote git repository, using a `Dockerfile` from `stdin`. The syntax uses the `-f` (or `--file`) option to specify the `Dockerfile` to use, using a hyphen (`-`) as filename to instruct `Docker` to read the `Dockerfile` from `stdin`:

    docker build [OPTIONS] -f- PATH

This syntax can be useful in situations where you want to build an image from a repository that does not contain a `Dockerfile`, or if you want to build with a custom `Dockerfile`, without maintaining your own fork of the repository.

The example below builds an image using a `Dockerfile` from `stdin`, and adds the `hello.c` file from the “hello-world” Git repository on GitHub.

    docker build -t myimage:latest -f- https://github.com/docker-library/hello-world.git <<EOF
    FROM busybox
    COPY hello.c .
    EOF

#### Exclude with `.dockerignore`

To exclude files not relevant to the build (without restructuring your source repository) use a `.dockerignore` file. This file supports exclusion patterns similar to `.gitignore` files. For information on creating one, see the [.dockerignore file](https://docs.docker.com/engine/reference/builder/#dockerignore-file).

#### Use multi-stage builds

[Multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/) allow you to drastically reduce the size of your final image, without struggling to reduce the number of intermediate layers and files.

Because an image is built during the final stage of the build process, you can minimize image layers by [leveraging build cache](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#leverage-build-cache).

For example, if your build contains several layers, you can order them from the less frequently changed (to ensure the build cache is reusable) to the more frequently changed:

* Install tools you need to build your application
* Install or update library dependencies
* Generate your application

#### Don’t install unnecessary packages

To reduce complexity, dependencies, file sizes, and build times, avoid installing extra or unnecessary packages just because they might be “nice to have.” For example, you don’t need to include a text editor in a database image.

#### Decouple applications

Each container should have only one concern. Decoupling applications into multiple containers makes it easier to scale horizontally and reuse containers. For instance, a web application stack might consist of three separate containers, each with its own unique image, to manage the **web application**, **database**, and an **in-memory cache** in a decoupled manner.

Limiting each container to one process is a good rule of thumb, but it is not a hard and fast rule. For example, not only can containers be spawned with an init process, some programs might spawn additional processes of their own accord. For instance, Celery can spawn multiple worker processes, and Apache can create one process per request.

Use your best judgment to keep containers as **clean** and **modular** as possible. If containers depend on each other, you can use [Docker container networks](https://docs.docker.com/engine/userguide/networking/) to ensure that these containers can communicate.

#### Minimize the number of layers

In **older versions** of `Docker`, it was important that you minimized the number of layers in your images to ensure they were performant. The following features were added to **reduce this limitation**:

* Only the instructions `RUN`, `COPY`, `ADD` create layers. Other instructions create temporary intermediate images, and do not increase the size of the build.
* Where possible, use multi-stage builds, and only copy the artifacts you need into the final image. This allows you to include tools and debug information in your intermediate build stages without increasing the size of the final image.

#### Sort multi-line arguments

Whenever possible, ease later changes by sorting multi-line arguments alphanumerically. This helps to avoid duplication of packages and make the list much easier to update. This also makes PRs a lot easier to read and review. Adding a space before a backslash (`\`) helps as well.

Here’s an example from the buildpack-deps image:

    RUN apt-get update && apt-get install -y \
      bzr \
      cvs \
      git \
      mercurial \
      subversion

#### Leverage build cache

When building an image, `Docker` steps through the instructions in your `Dockerfile`, executing each in the order specified. As each instruction is examined, `Docker` looks for an existing image in its cache that it can reuse, rather than creating a new (duplicate) image.

If you do not want to use the cache at all, you can use the `--no-cache=true` option on the docker build command. However, if you do let Docker use its cache, it is important to understand when it can, and cannot, find a matching image. The basic rules that Docker follows are outlined below:

* Starting with a parent image that is already in the cache, the next instruction is compared against all child images derived from that base image to see if one of them was built using the exact same instruction. If not, the cache is invalidated.

* In most cases, simply comparing the instruction in the Dockerfile with one of the child images is sufficient. However, certain instructions require more examination and explanation.

* For the `ADD` and `COPY` instructions, the contents of the file(s) in the image are examined and a checksum is calculated for each file. The last-modified and last-accessed times of the file(s) are not considered in these checksums. During the cache lookup, the checksum is compared against the checksum in the existing images. If anything has changed in the file(s), such as the contents and metadata, then the cache is invalidated.

* Aside from the `ADD` and `COPY` commands, cache checking does not look at the files in the container to determine a cache match. For example, when processing a `RUN apt-get -y update` command the files updated in the container are not examined to determine if a cache hit exists. In that case just the command string itself is used to find a match.

Once the cache is invalidated, all subsequent Dockerfile commands generate new images and the cache is not used.

### Dockerfile instructions

#### `FROM`

Whenever possible, use current official images as the basis for your images. We recommend the [Alpine image](https://hub.docker.com/_/alpine/) as it is tightly controlled and small in size (currently under 5 MB), while still being a full Linux distribution.

#### `LABEL`

You can add labels to your image to help organize images by project, record licensing information, to aid in automation, or for other reasons. For each label, add a line beginning with LABEL and with one or more key-value pairs. The following examples show the different acceptable formats. Explanatory comments are included inline.

Strings with spaces must be quoted or the spaces must be escaped. Inner quote characters ("), must also be escaped.

    # Set one or more individual labels
    LABEL com.example.version="0.0.1-beta"
    LABEL vendor1="ACME Incorporated"
    LABEL vendor2=ZENITH\ Incorporated
    LABEL com.example.release-date="2015-02-12"
    LABEL com.example.version.is-production=""

#### `RUN`

Split long or complex `RUN` statements on multiple lines separated with backslashes to make your `Dockerfile` more readable, understandable, and maintainable.

##### `apt-get`

Probably the most common use-case for `RUN` is an application of `apt-get`. Because it installs packages, the `RUN apt-get` command has several **gotchas** to look out for.

Avoid `RUN apt-get upgrade` and `dist-upgrade`, as many of the “essential” packages from the parent images cannot upgrade inside an **unprivileged container**. If a package contained in the parent image is out-of-date, contact its maintainers. If you know there is a particular package, foo, that needs to be updated, use `apt-get install -y foo` to update automatically.

Always combine `RUN apt-get update` with `apt-get install` in the same `RUN` statement. For example:

    RUN apt-get update && apt-get install -y \
        package-bar \
        package-baz \
        package-foo

Using `apt-get update` alone in a `RUN` statement causes caching issues and subsequent `apt-get install` instructions fail. For example, say you have a `Dockerfile`:

    FROM ubuntu:18.04
    RUN apt-get update
    RUN apt-get install -y curl

After building the image, all layers are in the Docker cache. Suppose you later modify `apt-get install` by adding extra package:

    FROM ubuntu:18.04
    RUN apt-get update
    RUN apt-get install -y curl nginx

Docker sees the initial and modified instructions as identical and reuses the cache from previous steps. As a result the `apt-get update` is **not executed** because the build uses the cached version. Because the `apt-get update` is not run, your build can potentially get an outdated version of the `curl` and `nginx` packages.

Using `RUN apt-get update && apt-get install -y` ensures your `Dockerfile` installs the latest package versions with no further coding or manual intervention. This technique is known as “cache busting”. You can also achieve cache-busting by specifying a package version. This is known as version pinning, for example:

    RUN apt-get update && apt-get install -y \
        package-bar \
        package-baz \
        package-foo=1.3.*

Version pinning forces the build to retrieve a particular version regardless of what’s in the cache. This technique can also reduce failures due to unanticipated changes in required packages.

Below is a well-formed `RUN` instruction that demonstrates all the `apt-get` recommendations.

    RUN apt-get update && apt-get install -y \
        aufs-tools \
        automake \
        build-essential \
        curl \
        dpkg-sig \
        libcap-dev \
        libsqlite3-dev \
        mercurial \
        reprepro \
        ruby1.9.1 \
        ruby1.9.1-dev \
        s3cmd=1.1.* \
     && rm -rf /var/lib/apt/lists/*

The `s3cmd` argument specifies a version `1.1.*`. If the image previously used an older version, specifying the new one causes a cache bust of `apt-get update` and ensures the installation of the new version. Listing packages on each line can also prevent mistakes in package duplication.

In addition, when you clean up the apt cache by removing `/var/lib/apt/lists` it reduces the image size, since the apt cache is not stored in a layer. Since the `RUN` statement starts with `apt-get update`, the package cache is always refreshed prior to `apt-get install`.

##### Using pipes

Some `RUN` commands depend on the ability to pipe the output of one command into another, using the pipe character (`|`), as in the following example:

`RUN wget -O - https://some.site | wc -l > /number`
Docker executes these commands using the `/bin/sh -c` interpreter, which only evaluates the exit code of the last operation in the pipe to determine success. In the example above this build step succeeds and produces a new image so long as the `wc -l` command succeeds, even if the `wget` command fails.

If you want the command to fail due to an error at any stage in the pipe, prepend set `-o pipefail &&` to ensure that an unexpected error prevents the build from inadvertently succeeding. For example:

    RUN set -o pipefail && wget -O - https://some.site | wc -l > /number

##### `CMD`

The `CMD` instruction should be used to **run the software** contained by your image, along with any arguments. `CMD` should almost always be used in the form of `CMD ["executable", "param1", "param2"…]`. Thus, if the image is for a service, such as Apache and Rails, you would run something like `CMD ["apache2","-DFOREGROUND"]`. Indeed, this form of the instruction is recommended for any service-based image.

In most other cases, `CMD` should be given an interactive shell, such as `bash`, `python` and `perl`. For example, `CMD ["perl", "-de0"]`, `CMD ["python"]`, or `CMD ["php", "-a"]`. Using this form means that when you execute something like `docker run -it python`, you’ll get dropped into a usable shell, ready to go. `CMD` **should rarely** be used in the manner of `CMD ["param", "param"]` in conjunction with `ENTRYPOINT`, unless you and your expected users are already quite familiar with how `ENTRYPOINT` works.

##### `EXPOSE`

The `EXPOSE` instruction indicates the ports on which a container listens for connections. Consequently, you should use the common, traditional port for your application. For example, an image containing the `Apache web server` would use `EXPOSE 80`, while an image containing `MongoDB` would use `EXPOSE 27017` and so on.

For external access, your users can execute docker run with a flag indicating how to map the specified port to the port of their choice. For container linking, Docker provides environment variables for the path from the recipient container back to the source (ie, `MYSQL_PORT_3306_TCP`).

##### `ENV`

To make new software easier to run, you can use `ENV` to update the `PATH` environment variable for the software your container installs. For example, `ENV PATH /usr/local/nginx/bin:$PATH` ensures that `CMD ["nginx"]` just works.

The `ENV` instruction is also useful for providing required environment variables specific to services you wish to containerize, such as Postgres’s `PGDATA`.

Lastly, `ENV` can also be used to set commonly used version numbers so that version bumps are easier to maintain, as seen in the following example:

    ENV PG_MAJOR 9.3
    ENV PG_VERSION 9.3.4
    RUN curl -SL http://example.com/postgres-$PG_VERSION.tar.xz | tar -xJC /usr/src/postgress && …
    ENV PATH /usr/local/postgres-$PG_MAJOR/bin:$PATH

Similar to having constant variables in a program (as opposed to hard-coding values), this approach lets you change a single `ENV` instruction to auto-magically bump the version of the software in your container.

Each `ENV` line creates a **new intermediate layer**, just like `RUN` commands. This means that even if you unset the environment variable in a future layer, it still persists in this layer and its value can be dumped. You can test this by creating a `Dockerfile` like the following, and then building it.

    FROM alpine
    ENV ADMIN_USER="mark"
    RUN echo $ADMIN_USER > ./mark
    RUN unset ADMIN_USER

    $ docker run --rm test sh -c 'echo $ADMIN_USER'

    mark

To prevent this, and really unset the environment variable, use a RUN command with shell commands, to `set`, use, and `unset` the variable **all in a single layer**. You can separate your commands with `;` or `&&`. If you use the second method, and one of the commands fails, the docker build also fails. This is usually a good idea. Using `\` as a line continuation character for Linux Dockerfiles improves readability. You could also put all of the commands into a shell script and have the `RUN` command just run that shell script.

    FROM alpine
    RUN export ADMIN_USER="mark" \
        && echo $ADMIN_USER > ./mark \
        && unset ADMIN_USER
    CMD sh
    
    $ docker run --rm test sh -c 'echo $ADMIN_USER'

##### 'ADD or COPY'

Although `ADD` and `COPY` are functionally similar, generally speaking, `COPY` is preferred. That’s because it’s more transparent than `ADD`. `COPY` only supports the **basic copying** of **local files** into the container, while `ADD` has some features (like local-only `tar` extraction and remote `URL` support) that are not immediately obvious. Consequently, the best use for `ADD` is **local tar file auto-extraction** into the image, as in `ADD rootfs.tar.xz /`.

If you have multiple `Dockerfile` steps that use different files from your context, `COPY` them **individually**, rather than all at once. This ensures that each step’s build cache is only invalidated (forcing the step to be re-run) if the specifically required files change.
For example:

    COPY requirements.txt /tmp/
    RUN pip install --requirement /tmp/requirements.txt
    COPY . /tmp/

Results in fewer cache invalidations for the `RUN` step, than if you put the `COPY . /tmp/` before it.

Because image size matters, using `ADD` to fetch packages from remote `URLs` is strongly discouraged; you should use `curl` or `wget` instead. That way you can delete the files you no longer need after they’ve been extracted and you don’t have to add another layer in your image. For example, you should **avoid** doing things like:

    ADD http://example.com/big.tar.xz /usr/src/things/
    RUN tar -xJf /usr/src/things/big.tar.xz -C /usr/src/things
    RUN make -C /usr/src/things all

And instead, do something like:

    RUN mkdir -p /usr/src/things \
        && curl -SL http://example.com/big.tar.xz \
        | tar -xJC /usr/src/things \
        && make -C /usr/src/things all

For other items (files, directories) that do not require ADD’s tar auto-extraction capability, you should always use `COPY`.

##### `ENTRYPOINT`

The best use for `ENTRYPOINT` is to set the image’s main command, allowing that image to be run as though it was that command (and then use `CMD` as the default flags).
Let’s start with an example of an image for the command line tool `s3cmd`:

    ENTRYPOINT ["s3cmd"]
    CMD ["--help"]

Now the image can be run like this to show the command’s help:

    $ docker run s3cmd

Or using the right parameters to execute a command:

    $ docker run s3cmd ls s3://mybucket

##### `VOLUME`

The `VOLUME` instruction should be used to expose any database storage area, configuration storage, or files/folders created by your docker container. You are strongly encouraged to use `VOLUME` for any mutable and/or user-serviceable parts of your image.

##### `USER`

If a service can run without privileges, use `USER` to change to a non-root user. Start by creating the user and group in the `Dockerfile` with something like `RUN groupadd -r postgres && useradd --no-log-init -r -g postgres postgres`.

Avoid installing or using `sudo` as it has unpredictable `TTY` and signal-forwarding behavior that can cause problems. If you absolutely need functionality similar to sudo, such as initializing the daemon as root but running it as non-root, consider using “`gosu`”.
Lastly, to reduce layers and complexity, **avoid switching USER** back and forth frequently.

##### `WORKDIR`

For clarity and reliability, you should **always use absolute paths for your `WORKDIR`**. Also, you should use `WORKDIR` instead of proliferating instructions like `RUN cd … && do-something`, which are **hard to read**, troubleshoot, and maintain**.

##### `ONBUILD`

An `ONBUILD` command executes after the current `Dockerfile` build completes. `ONBUILD` executes in any child image derived `FROM` the current image. Think of the `ONBUILD` command as an instruction the parent `Dockerfile` gives to the child `Dockerfile`.

A Docker build executes `ONBUILD` commands before any command in a child `Dockerfile`.

## [Manage data in Docker](https://docs.docker.com/storage/)

By default all files created inside a container are stored on a **writable container layer**. This means that:

* The data **doesn’t persist** when that container no longer exists, and it can be **difficult to get the data out** of the container if another process needs it.

* A container’s writable layer is tightly **coupled to the host machine** where the container is running. You can’t easily move the data somewhere else.

* Writing into a container’s writable layer requires a storage driver to manage the filesystem. The storage driver provides a union filesystem, using the Linux kernel. This extra abstraction **reduces performance** as compared to using data volumes, which write directly to the host filesystem.

### Choose the right type of mount

* **Volumes** are stored in a part of the host filesystem which is **managed by Docker** (`/var/lib/docker/volumes/` on Linux). Non-Docker processes should not modify this part of the filesystem. **Volumes are the best way to persist data in Docker**.

* **Bind mounts** may be stored anywhere on the host system. They may even be important system files or directories. Non-Docker processes on the Docker host or a Docker container can modify them at any time.

* tmpfs mounts are stored in the host system’s memory only, and are never written to the host system’s filesystem.

### More details about mount types

* **Volumes**: Created and managed by Docker. You can create a volume **explicitly** using the `docker volume create` command, or Docker can create a volume **during container or service creation**.

When you create a volume, it is stored within a directory on the Docker host. When you mount the volume into a container, this directory is what is mounted into the container. This is **similar to the way that bind mounts** work, except that volumes are **managed by Docker** and are **isolated** from the core functionality of the host machine.

A given volume can be **mounted into multiple containers** simultaneously. When no running container is using a volume, the volume is still available to Docker and is **not removed automatically**. You can **remove unused volumes** using `docker volume prune`.

When you mount a volume, it may be **named** or **anonymous**. Anonymous volumes are not given an explicit name when they are first mounted into a container, so Docker gives them a random name that is guaranteed to be **unique within a given Docker host**. Besides the name, named and anonymous volumes behave in the same ways.

Volumes also support the use of **volume drivers**, which allow you to store your data on **remote hosts** or **cloud providers**, among other possibilities.

* **Bind mounts**: Available since the early days of Docker. Bind mounts have limited functionality compared to volumes. When you use a bind mount, a file or directory on the host machine is mounted into a container. The file or directory is **referenced by its full path** on the host machine. The file or directory does **not need to exist on the Docker host already**. It is **created on demand** if it does not yet exist. Bind mounts are very **performant**, but they rely on the host machine’s filesystem having a **specific directory structure available**. If you are developing new Docker applications, consider using named volumes instead. You can’t use Docker CLI commands to directly manage bind mounts.

One **side effect** of using bind mounts, for better or for worse, is that you **can change the host filesystem** via processes running in a container, including creating, modifying, or deleting important system files or directories. This is a powerful ability which can have **security implications**, including impacting non-Docker processes on the host system.

* **tmpfs mounts**: A tmpfs mount is not persisted on disk, either on the Docker host or within a container. It can be used by a container during the lifetime of the container, to store non-persistent state or sensitive information. For instance, internally, swarm services use tmpfs mounts to mount secrets into a service’s containers.

* **named pipes**: An npipe mount can be used for communication between the Docker host and a container. Common use case is to run a third-party tool inside of a container and connect to the Docker Engine API using a named pipe.

Bind mounts and volumes can both be mounted into containers using the `-v` or `--volume` flag, but the syntax for each is slightly different. For `tmpfs` mounts, you can use the `--tmpfs` flag. However, in Docker 17.06 and higher, we **recommend using** the `--mount` flag for both containers and services, for bind mounts, volumes, or `tmpfs` mounts, as the syntax is more clear.

### Good use cases for volumes

Volumes are the preferred way to persist data in Docker containers and services. Some use cases for volumes include:

* **Sharing data among multiple running containers**. If you don’t explicitly create it, a volume is created the first time it is mounted into a container. When that container stops or is removed, the volume still exists. **Multiple containers can mount the same volume simultaneously**, either **read-write** or **read-only**. Volumes are only removed when you explicitly remove them.

* When the Docker host is not guaranteed to have a given directory or file structure. Volumes help you decouple the configuration of the Docker host from the container runtime.

* When you want to store your container’s data on a remote host or a cloud provider, rather than locally.

* When you need to **back up**, **restore**, or **migrate** data from one Docker host to another, volumes are a better choice. You can stop containers using the volume, then back up the volume’s directory (such as `/var/lib/docker/volumes/<volume-name>`).

### Good use cases for bind mounts

In general, you should **use volumes where possible**. Bind mounts are appropriate for the following types of use case:

* **Sharing configuration files** from the host machine to containers. This is how Docker provides DNS resolution to containers by default, by mounting `/etc/resolv.conf` from the host machine into each container.

* **Sharing source code** or **build artifacts** between a development environment on the Docker host and a container. For instance, you may mount a Maven `target/` directory into a container, and each time you build the Maven project on the Docker host, the container gets access to the rebuilt artifacts.

If you use Docker for development this way, your **production `Dockerfile`** would copy the production-ready artifacts directly into the image, rather than relying on a bind mount.

* When the file or directory structure of the Docker host is guaranteed to be consistent with the bind mounts the containers require.

### Good use cases for tmpfs mounts

**tmpfs** mounts are best used for cases when you do **not want the data to persist** either on the host machine or within the container. This may be for **security reasons** or to **protect the performance** of the container when your application needs to write a large volume of non-persistent state data.

### Tips for using bind mounts or volumes

If you use either bind mounts or volumes, keep the following in mind:

* If you mount an empty volume into a directory in the container in which files or directories exist, these files or directories are propagated (copied) into the volume. Similarly, if you start a container and specify a volume which does not already exist, an empty volume is created for you. This is a good way to **pre-populate** data that another container needs.

* If you mount a bind mount or non-empty volume into a directory in the container in which some files or directories exist, these files or directories are **obscured** by the mount, just as if you saved files into /mnt on a Linux host and then mounted a USB drive into /mnt. The contents of /mnt would be obscured by the contents of the USB drive until the USB drive were unmounted. **The obscured files are not removed or altered, but are not accessible while the bind mount or volume is mounted**.

## [Use volumes](https://docs.docker.com/storage/volumes/)

**Volumes are the preferred mechanism for persisting data generated by and used by Docker containers**. While bind mounts are dependent on the directory structure of the host machine, volumes are **completely managed by Docker**. Volumes have several advantages over bind mounts:

* Volumes are **easier to back up or migrate** than bind mounts.
* You can manage volumes using Docker CLI commands or the Docker API.
* Volumes work on both Linux and Windows containers.
* Volumes can be more safely shared among multiple containers.
* Volume drivers let you store volumes on **remote hosts** or **cloud providers**, to **encrypt the contents** of volumes, or to **add other functionality**.
* New volumes can have their content pre-populated by a container.

In addition, volumes are often a better choice than persisting data in a container’s writable layer, because a volume does not increase the size of the containers using it, and the volume’s contents exist outside the lifecycle of a given container.

If your container generates non-persistent state data, consider using a tmpfs mount to avoid storing the data anywhere permanently, and to increase the container’s performance by avoiding writing into the container’s writable layer.

Volumes use `rprivate` bind propagation, and bind propagation is not configurable for volumes.

### Choose the -v or --mount flag

### Create and manage volumes

Create a volume:

    $ docker volume create my-vol
    List volumes:
    $ docker volume ls

    local               my-vol

Inspect a volume:

    $ docker volume inspect my-vol
    [
        {
            "Driver": "local",
            "Labels": {},
            "Mountpoint": "/var/lib/docker/volumes/my-vol/_data",
            "Name": "my-vol",
            "Options": {},
            "Scope": "local"
        }
    ]

Remove a volume:

    $ docker volume rm my-vol

### Start a container with a volume

If you start a container with a volume that does not yet exist, Docker creates the volume for you. The following example mounts the volume `myvol2` into `/app/` in the container.

    $ docker run -d \
      --name devtest \
      --mount source=myvol2,target=/app \
      nginx:latest

Use `docker inspect devtest` to verify that the volume was created and mounted correctly.

Stop the container and remove the volume. Note volume removal is a separate step.

    $ docker container stop devtest

    $ docker container rm devtest

    $ docker volume rm myvol2

### Share data among machines

When building fault-tolerant applications, you might need to **configure multiple replicas of the same service to have access to the same files**.

There are several ways to achieve this when developing your applications. One is to add logic to your application to store files on a cloud object storage system like Amazon S3. Another is to **create volumes with a driver that supports writing files to an external storage system** like NFS or Amazon S3.

Volume drivers allow you to **abstract the underlying storage system from the application logic**. For example, if your services use a volume with an NFS driver, you can update the services to use a different driver, as an example to store data in the cloud, without changing the application logic.

### Use a volume driver

When you create a volume using `docker volume create`, or when you start a container which uses a not-yet-created volume, you can specify a volume driver. The following examples use the `vieux/sshfs` volume driver, first when creating a standalone volume, and then when starting a container which creates a new volume.

#### Initial set-up

This example assumes that you have two nodes, the first of which is a Docker host and can connect to the second using SSH.

On the Docker host, install the `vieux/sshfs` plugin:

    $ docker plugin install --grant-all-permissions vieux/sshfs

#### Create a volume using a volume driver

This example specifies a SSH password, but if the two hosts have shared keys configured, you can omit the password. Each volume driver may have zero or more configurable options, each of which is specified using an `-o` flag.

    $ docker volume create --driver vieux/sshfs \
      -o sshcmd=test@node2:/home/test \
      -o password=testpassword \
      sshvolume

#### Start a container which creates a volume using a volume driver

This example specifies a SSH password, but if the two hosts have shared keys configured, you can omit the password. Each volume driver may have zero or more configurable options. **If the volume driver requires you to pass options, you must use the `--mount` flag to mount the volume, rather than `-v`**.

    $ docker run -d \
      --name sshfs-container \
      --volume-driver vieux/sshfs \
      --mount src=sshvolume,target=/app,volume-opt=sshcmd=test@node2:/home/test,volume-opt=password=testpassword \
      nginx:latest

#### Create a service which creates an NFS volume

This example shows how you can create an NFS volume when creating a service. This example uses `10.0.0.10` as the NFS server and `/var/docker-nfs` as the exported directory on the NFS server. Note that the volume driver specified is local.

NFSv3

    $ docker service create -d \
      --name nfs-service \
      --mount 'type=volume,source=nfsvolume,target=/app,volume-driver=local,volume-opt=type=nfs,volume-opt=device=:/var/docker-nfs,volume-opt=o=addr=10.0.0.10' \
      nginx:latest

NFSv4

    docker service create -d \
        --name nfs-service \
        --mount 'type=volume,source=nfsvolume,target=/app,volume-driver=local,volume-opt=type=nfs,volume-opt=device=:/,"volume-opt=o=10.0.0.10,rw,nfsvers=4,async"' \
        nginx:latest

### Backup, restore, or migrate data volumes

Volumes are useful for backups, restores, and migrations. Use the `--volumes-from` flag to create a new container that mounts that volume.

#### Backup a container

For example, create a new container named `dbstore`:

    $ docker run -v /dbdata --name dbstore ubuntu /bin/bash

Then in the next command, we:

* Launch a new container and mount the volume from the dbstore container
* Mount a local host directory as `/backup`
* Pass a command that tars the contents of the dbdata volume to a `backup.tar` file inside our `/backup` directory.

    $ docker run --rm --volumes-from dbstore -v $(pwd):/backup ubuntu tar cvf /backup/backup.tar /dbdata

When the command completes and the container stops, we are left with a backup of our `dbdata` volume.

#### Restore container from backup

With the backup just created, you can restore it to the same container, or another that you made elsewhere.
For example, create a new container named `dbstore2`:

    $ docker run -v /dbdata --name dbstore2 ubuntu /bin/bash

Then un-tar the backup file in the new container`s data volume:

    $ docker run --rm --volumes-from dbstore2 -v $(pwd):/backup ubuntu bash -c "cd /dbdata && tar xvf /backup/backup.tar --strip 1"

You can use the techniques above to automate backup, migration and restore testing using your preferred tools.

### Remove volumes

A Docker data volume persists after a container is deleted. There are two types of volumes to consider:

* **Named volumes** have a specific source from outside the container, for example `awesome:/bar`.

* **Anonymous volumes** have no specific source so when the container is deleted, instruct the Docker Engine daemon to remove them.

#### Remove anonymous volumes

To automatically remove anonymous volumes, use the `--rm` option. For example, this command creates an anonymous `/foo` volume. When the container is removed, the Docker Engine removes the `/foo` volume but not the awesome volume.

    $ docker run --rm -v /foo -v awesome:/bar busybox top

#### Remove all volumes

To remove all unused volumes and free up space:

    $ docker volume prune

## [Use bind mounts](https://docs.docker.com/storage/bind-mounts/)

Bind mounts have been around since the early days of Docker. Bind mounts have limited functionality compared to volumes. When you use a bind mount, a file or directory on the host machine is mounted into a container. The file or directory is referenced by its full or relative path on the host machine. By contrast, when you use a volume, a new directory is created within Docker’s storage directory on the host machine, and Docker manages that directory’s contents.

The file or directory does **not need to exist** on the Docker host already. It is **created on demand** if it does not yet exist. Bind mounts are very performant, but they rely on the host machine’s filesystem having a specific directory structure available. If you are developing new Docker applications, consider using named volumes instead. You can’t use Docker CLI commands to directly manage bind mounts.

### Start a container with a bind mount

Consider a case where you have a directory `source` and that when you build the source code, the artifacts are saved into another directory, `source/target/`. You want the artifacts to be available to the container at `/app/`, and you want the container to get access to a new build each time you build the source on your development host. Use the following command to bind-mount the `target/` directory into your container at `/app/`. Run the command from within the `source` directory. The `$(pwd)` sub-command expands to the current working directory on Linux or macOS hosts.

    $ docker run -d \
      -it \
      --name devtest \
      --mount type=bind,source="$(pwd)"/target,target=/app \
      nginx:latest

Stop the container:

    $ docker container stop devtest

    $ docker container rm devtest

#### Mount into a non-empty directory on the container

If you bind-mount into a non-empty directory on the container, the directory’s existing contents are **obscured by the bind mount**. This can be beneficial, such as when you want to **test a new version of your application without building a new image**. However, it can also be surprising and this behavior differs from that of docker volumes.

    $ docker run -d \
      -it \
      --name broken-container \
      --mount type=bind,source=/tmp,target=/usr \
      nginx:latest

    docker: Error response from daemon: oci runtime error: container_linux.go:262:
    starting container process caused "exec: \"nginx\": executable file not found in $PATH".

The container is created but does not start. Remove it:

    $ docker container rm broken-container

### Use a read-only bind mount

This example modifies the one above but mounts the directory as a **read-only** bind mount, by adding `ro` to the (empty by default) list of options, after the mount point within the container. Where multiple options are present, separate them by commas.

    $ docker run -d \
      -it \
      --name devtest \
      --mount type=bind,source="$(pwd)"/target,target=/app,readonly \
      nginx:latest

Stop the container:

    $ docker container stop devtest

    $ docker container rm devtest

### Configure bind propagation

Bind propagation defaults to `rprivate` for both bind mounts and volumes. It is only configurable for bind mounts, and only on Linux host machines. Bind propagation is an advanced topic and many users never need to configure it.
