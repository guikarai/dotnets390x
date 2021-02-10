# Running a dotnet application s390x
Thanks to Mono work there is an opportunity to run some C# application on linux on IBM Z and LinuxONE, and so z/OS Container Extensions aka. zCX.

Mono is the open source development platform based on the .NET Framework. It allows developers to build cross-platform applications with improved developer productivity. Mono’s .NET implementation is based on the ECMA standards for C# and the Common Language Infrastructure.
More information about [Mono](https://www.mono-project.com/docs/about-mono/).

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
* **nuget :** NuGet is the package manager for .NET. The NuGet client tools provide the ability to produce and consume packages.
* **vi :** Editor to open and edit file. Helpful for creating additional c# project if needed.

3 files are copied in the images. These files are C# helloworld like application that can be executed from a

As you can see, we are exposing the port 8080 so that a C# can deliver Web services using this port.

```
FROM ubuntu:20.04
ENV MONO_VERSION 6.10.0.104
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
  && apt-get install -y mono-runtime mono-devel \
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
## 
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

# Simple Helloworld

# Simple Web Helloworld

# Conclusions
