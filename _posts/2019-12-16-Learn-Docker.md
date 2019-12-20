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
