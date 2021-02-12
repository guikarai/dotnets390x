# Running a dotnet application s390x
Thanks to Mono work there is an opportunity to run some C# applications on linux on IBM Z and LinuxONE, and also on z/OS Container Extensions aka. zCX.

## What is mono?

Mono is the open source development platform based on the .NET Framework. It allows developers to build cross-platform applications with improved developer productivity. Mono's .NET implementation is based on the ECMA standards for C# and the Common Language Infrastructure.
More information about [Mono](https://www.mono-project.com/docs/about-mono/).

## Why mono?

Mono offers some interesting features:
* **Binary compatibility:** you can import CLR compliant executables or libraries, just add them as a reference into your Mono project and directly use them without any modifications and vice-versa. This is possible because Mono was built on the implementation of the ECMA's Common Language Infrastructure.
* **Microsoft Compatible API:** major .NET Framework components (ASP.NET, ADO.NET, Windows Forms) can run without recompilation.
* **C# from 1.0 to 5.0 full feature-complete** ( full support of Linq, dynamic, ...).
* Mono SDK is available on Windows, Linux, OSX, BSD, ...
* Available in x86, x64, ARM, power pc, and **S390x architectures**

However, there are some compatibility limitations which are:
* Windows Presentation Foundation (WPF), Windows Workflow Foundation (WWF) are not supported at all.
* Limited supports on Windows Communication Foundation (WCF).
* Limited supports on Asp.net 4.5 async task.
* MVC4 or MVC5, some features aren't working yet.

Mono-develop is a complete IDE offering nice editing capabilities. The Mono-develop IDE is also very intuitive. If you're okay with Visual Studio, working with Mono-develop is as well as simple too. The chart below shows utilities to build and run .NET programs according the targeted version:

|**Tools**|**.NET Framework**|**.NET Core**|**Mono**|
|---|---|---|--|
|**Build/Compile**|csc|dotnet build|mcs|
|**Run**|cs|dotnet|mono|

The following is about describing how to run simple C# application on s390x environment.

# s390x architecture
Mono supports the s390x architecture, it can run natively on:
* Linux on IBM Z
* Linux on LinuxONE

and it can also run in a Linux docker container on :
* z/OS Container Extension (zCX)
* Linux on IBM Z docker host
* Linux on LinuxONE docker host

For the following, we will focus on the Linux docker experience on s390x.

# Docker container to run a C# application
## Building the docker image

You can find below a Dockerfile used to build from an Ubuntu base image a mono environment plus several other packages:
* **mono :** There are several components that make up Mono: C# Compiler, Mono runtime, .NET Framework Class Library, Mono Class Library.
* **nuget :** [NuGet](https://www.nuget.org/) is the package manager for .NET. The NuGet client tools provide the ability to produce and consume packages.
* **vi :** Editor to open and edit file. Helpful for creating additional C# project if needed.

3 files have been copied to the docker image. These files are C# helloworld programs that can be compiled with mcs and executed with mono.

As you can see, we are exposing the port 8080 so that a C# program can deliver Web services using this port.

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
  && DEBIAN_FRONTEND=noninteractive apt-get install -y mono-runtime mono-devel \
  && apt-get install -y vim \
  && apt-get install -y nuget \
  && nuget update -self
# optional
#RUN mkdir /root/helloworld
#COPY *.cs /root/helloworld
#RUN mcs /root/helloworld/hello.cs
#RUN mcs /root/helloworld/helloform.cs
#RUN mcs /root/helloworld/helloweb.cs
WORKDIR /root
EXPOSE 8080
```

To build the image, run the following command:
```
docker build -f Dockerfile -t <image_name> .
```

Once build, to check that the image is OK, run the following command:
```
docker images
```

## Creating and connecting to the docker container

To create a docker container from the docker image you just built, issue the following command on the docker host:
```
docker run -it --name <container_name> -p 8080:8080 <image_name>  bash
```
You should now be running bash in the container.


To check that the container is running, please issue the following command on the docker host:
```
docker ps
CONTAINER ID        IMAGE                   COMMAND                  CREATED             STATUS              PORTS               NAMES
c6e811baad63        mono-nuget-s390x:6.10   "/bin/bash"              2 weeks ago         Up 3 days                               <container_name>
```

To reconnect to an existing running container, please issue the following command on the docker host:
```
docker exec -ti <container_name> /bin/bash
root@952993dc07ea:/#
```

Once connected, you can check that the environment is properly set. 
To do so, issue the following command to inspect the Mono configuration:
```
root@952993dc07ea:/# mono --version
Mono JIT compiler version 6.8.0.105 (Debian 6.8.0.105+dfsg-2 Wed Feb 26 23:24:30 UTC 2020)
Copyright (C) 2002-2014 Novell, Inc, Xamarin Inc and Contributors. www.mono-project.com
	TLS:           __thread
	SIGSEGV:       normal
	Notifications: epoll
	Architecture:  s390x
	Disabled:      none
	Misc:          softdebug 
	Interpreter:   yes
	Suspend:       preemptive
	GC:            sgen (concurrent by default)
```

Now let's inspect, build and run some simple C# application from there.

# Simple Helloworld
Move in /root/helloworld/ directory.
```
cd /root/helloworld/
```

Edit the file hello.cs using the vi command below. Feel free to edit the code to display a custom message.
```
vi hello.cs
```

Content looks like the following:
```
using System;

namespace HelloWorld
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello zCX World!");  //<----- Modify the text as you wish here.
        }
    }
}
```

Once edited, let's build this simple helloworld C# application.
To do so, please issue the following command:
```
mcs hello.cs
```

Listing the content of the current directory, you may see that a new file has been created: hello.exe
This file can be executed with mono:
```
mono hello.exe
Hello zCX World!
```
As you can see, the very simple C# application works. It displays the sentence "Hello zCX World!".

Now, let's push the bar higher with an hello world webapp.

# Simple Web helloworld
Now lets work on a C# helloworld webapp .
Still in the current directory, please edit the file named helloweb.cs with the following command:
```
vi helloweb.cs
```
If you want a look at the file content, please check [src/helloweb.cs](../master/src/helloweb.cs).
This C# helloworld webapp responds "Hello World" when receiving an HTTP GET /hello request on its listening port 8080.
To build this application, and to generate the executable binary, please issue the following command:
```
mcs helloweb.cs
```
As we did before, to use the executable binary, please issue the following command:
```
mono helloweb.exe
```

Open another terminal in the docker container or from the docker host, you can test if the web application works by issuing the following command:
```
curl -v http://localhost:8080/hello/
```
Output looks like the following:
```
*   Trying 127.0.0.1:8080...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET /hello/ HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.68.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Server: Mono-HTTPAPI/1.0
< Date: Thu, 11 Feb 2021 15:26:47 GMT
< Content-Length: 70
< Keep-Alive: timeout=15,max=100
< 
* Connection #0 to host localhost left intact
<HTML><BODY>Hello World, the time is 02/11/2021 15:26:47</BODY></HTML>root@c6e811baad63:/#
```
If the docker container has been started with -p 8080:8080, the request could also be sent from a different machine and use the IP address instead of localhost.

# Conclusions
