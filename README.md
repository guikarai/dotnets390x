# Running a dotnet application s390x
Thanks to Mono work there is an opportunity to run some C# application on linux on IBM Z and LinuxONE, and so z/OS Container Extensions aka. zCX.

Mono is the open source development platform based on the .NET Framework. It allows developers to build cross-platform applications with improved developer productivity. Mono’s .NET implementation is based on the ECMA standards for C# and the Common Language Infrastructure.
More information about [Mono](https://www.mono-project.com/docs/about-mono/).

Mono-develop is a complete IDE offering nice editing capabilities. The Mono-develop IDE is also very intuitive. If you’re ok with Visual Studio, working with Mono-develop is as well as simple too. The chart below shows utilities to build and run .NET programs according the targeted version:
|**Tools**|**.NET Framework**|**.NET Core**|**Mono**|
|**Build/Compile**|csc|dotnet build|mcs|
|**Run**|cs|dotnet|mono|

The following is about describing how to run simple C# application on s390x environment.

# Requirements
Mono supports the s390x architecture, and runs as well natively on:
* Linux on IBM Z
* Linux on LinuxONE

Alternatively, Mono supports the docker architecture and can run in Linux Docker container. This opening oportunity to run it on:
* z/OS Container Extension (zCX)
* Linux on IBM Z
* Linux on LinuxONE

For the following we will focus on the linux docker experience on s390x.

# Docker container running a dotnet application
## Building the docker image

You can find below a Dockerfile use to build from an Ubuntu base image a mono environment plus several other packages:
* **mono :** There are several components that make up Mono: C# Compiler, Mono runtime, .NET Framework Class Library, Mono Class Library.
* **nuget :** [NuGet](https://www.nuget.org/) is the package manager for .NET. The NuGet client tools provide the ability to produce and consume packages.
* **vi :** Editor to open and edit file. Helpful for creating additional c# project if needed.

3 files are copied in the images. These files are C# helloworld like application that can be executed from a

As you can see, we are exposing the port 8080 so that a C# can deliver Web services using this port.

```
FROM ubuntu:18.04

ENV MONO_VERSION 6.10.0.104
RUN DEBIAN_FRONTEND=noninteractive
RUN apt-get update 
RUN apt-get install -y --no-install-recommends gnupg dirmngr ca-certificates
RUN  rm -rf /var/lib/apt/lists/* \
  && export GNUPGHOME="$(mktemp -d)" 
RUN  gpg --batch --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF \
  && gpg --batch --export --armor 3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF > /etc/apt/trusted.gpg.d/mono.gpg.asc \
  && gpgconf --kill all \
  && rm -rf "$GNUPGHOME" \
  && apt-key list | grep Xamarin \
  && apt-get purge -y --auto-remove gnupg dirmngr
RUN echo "deb https://download.mono-project.com/repo/ubuntu stable-focal/snapshots/$MONO_VERSION main" > /etc/apt/sources.list.d/mono-official-stable.list \
  && apt-get update \
  && DEBIAN_FRONTEND=noninteractive apt-get install -y mono-runtime mono-devel \
  && apt-get install -y vim \
  && apt-get install -y nuget \
  && nuget update -self
RUN mkdir /root/helloworld
COPY *.cs /root/helloworld
RUN mcs /root/helloworld/hello.cs
RUN mcs /root/helloworld/helloform.cs
RUN mcs /root/helloworld/helloweb.cs
WORKDIR /root
EXPOSE 8080
```

To build the image, run the following command:
```
Docker build -f Dockerfile
```

Once build, to check that the image is OK, run the following command:
```
Docker images
```

## Creating and connecting to the docker container

To create a docker container from the just built image, please issue the following command:


To check that the container is running, please issue the following command:
```
root@ub20dpp:~# docker ps
CONTAINER ID        IMAGE                   COMMAND                  CREATED             STATUS              PORTS               NAMES
952993dc07ea        s390x/ubuntu:18.04      "bash"                   About an hour ago   Up About an hour                        optimistic_sutherland
```

To connect in the deployed container, please issue the following command:
```
root@ub20dpp:~# docker exec -ti 952993dc07ea /bin/bash
root@952993dc07ea:/#
```

Once connected, you can check that the environment is properly set. 
To do so, issue the following command to inspect the Mono configuration:

```
```

Now let's inspect, build and run some simple C# application from there.

# Simple Helloworld

# Simple Web Helloworld

# Conclusions
