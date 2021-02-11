# Running a dotnet application s390x
Thanks to Mono work there is an opportunity to run some C# application on linux on IBM Z and LinuxONE, and so z/OS Container Extensions aka. zCX.

## What is mono?

Mono is the open source development platform based on the .NET Framework. It allows developers to build cross-platform applications with improved developer productivity. Mono’s .NET implementation is based on the ECMA standards for C# and the Common Language Infrastructure.
More information about [Mono](https://www.mono-project.com/docs/about-mono/).

## Why mono?

Mono offers some interesting features:
* **Binary compatibility:** you can import CLR compliant executables or libraries, just add them as a reference into your Mono project and directly use them without any modifications and vice-versa. This is possible because Mono was built on the implementation of the ECMA’s Common Language Infrastructure.
* **Microsoft Compatible API:** major .NET Framework components (ASP.NET, ADO.NET, Windows Forms) can run without recompilation.
* **C# from 1.0 to 5.0 full feature-complete** ( full support of Linq, dynamic,…).
* Mono SDK is available on Windows, Linux, OSX, BSD,…
* Available in x86, x64, ARM, power pc, and **S390x architectures**

However, there are some compatibility limitations which are:
* Windows Presentation Foundation (WPF), Windows Workflow Foundation (WWF) are not supported at all.
* Limited supports on Windows Communication Foundation (WCF).
* Limited supports on Asp.net 4.5 async task.
* MVC4 or MVC5, some features aren’t working yet.

Mono-develop is a complete IDE offering nice editing capabilities. The Mono-develop IDE is also very intuitive. If you’re ok with Visual Studio, working with Mono-develop is as well as simple too. The chart below shows utilities to build and run .NET programs according the targeted version:

|**Tools**|**.NET Framework**|**.NET Core**|**Mono**|
|---|---|---|--|
|**Build/Compile**|csc|dotnet build|mcs|
|**Run**|cs|dotnet|mono|

The following is about describing how to run simple C# application on s390x environment.

# Requirements
Mono supports the s390x architecture, and runs as well natively on:
* Linux on IBM Z
* Linux on LinuxONE

Alternatively, Mono supports the docker architecture and can run in Linux Docker container. This opening oportunity to run it on:
* z/OS Container Extension (zCX)
* Linux on IBM Z docker host
* Linux on LinuxONE docker host

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
docker ps
CONTAINER ID        IMAGE                   COMMAND                  CREATED             STATUS              PORTS               NAMES
c6e811baad63        mono-nuget-s390x:6.10   "/bin/bash"              2 weeks ago         Up 3 days                               magical_wozniak
```

To connect in the deployed container, please issue the following command:
```
docker exec -ti c6e811baad63 /bin/bash
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

Edit the file hello.cs using the vi command below. Feel free to edit text to be displayed as you wish.
```
vi hello.cs
```

Content to look like the following:
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
This file can be executed with mono thanks to the following command:
```
mono hello.exe
Hello zCX World!
```
As you can see, the very simple C# application works. It displays the sentense "Hello zCX World!".

Now, let's push higher the bar with an hello world web-app.

# Simple Web Helloworld
Now lest work on a C# webapp helloworld application.
Still in the current directory, please edit the file named helloweb.cs with the following command:
```
vi helloweb.cs
```
Content to look like the following:
```
using System;
using System.Collections.Generic;
using System.Net;
using System.Text;
using System.Threading;
 
namespace WebServer
{
   public class WebServer
   {
      private readonly HttpListener _listener = new HttpListener();
      private readonly Func<HttpListenerRequest, string> _responderMethod;
 
      public WebServer(IReadOnlyCollection<string> prefixes, Func<HttpListenerRequest, string> method)
      {
         if (!HttpListener.IsSupported)
         {
            throw new NotSupportedException("Needs Windows XP SP2, Server 2003 or later.");
         }
             
         // URI prefixes are required eg: "http://localhost:8080/test/"
         if (prefixes == null || prefixes.Count == 0)
         {
            throw new ArgumentException("URI prefixes are required");
         }
         
         if (method == null)
         {
            throw new ArgumentException("responder method required");
         }
 
         foreach (var s in prefixes)
         {
            _listener.Prefixes.Add(s);
         }
 
         _responderMethod = method;
         _listener.Start();
      }
 
      public WebServer(Func<HttpListenerRequest, string> method, params string[] prefixes)
         : this(prefixes, method)
      {
      }
 
      public void Run()
      {
         ThreadPool.QueueUserWorkItem(o =>
         {
            Console.WriteLine("Webserver running...");
            try
            {
               while (_listener.IsListening)
               {
                  ThreadPool.QueueUserWorkItem(c =>
                  {
                     var ctx = c as HttpListenerContext;
                     try
                     {
                        if (ctx == null)
                        {
                           return;
                        }
                            
                        var rstr = _responderMethod(ctx.Request);
                        var buf = Encoding.UTF8.GetBytes(rstr);
                        ctx.Response.ContentLength64 = buf.Length;
                        ctx.Response.OutputStream.Write(buf, 0, buf.Length);
                     }
                     catch
                     {
                        // ignored
                     }
                     finally
                     {
                        // always close the stream
                        if (ctx != null)
                        {
                           ctx.Response.OutputStream.Close();
                        }
                     }
                  }, _listener.GetContext());
               }
            }
            catch (Exception ex)
            {
               // ignored
            }
         });
      }
 
      public void Stop()
      {
         _listener.Stop();
         _listener.Close();
      }
   }
 
   internal class Program
   {
      public static string SendResponse(HttpListenerRequest request)
      {
         return string.Format("<HTML><BODY>Hello World, the time is {0}</BODY></HTML>", DateTime.Now);
      }
 
      private static void Main(string[] args)
      {
         var ws = new WebServer(SendResponse, "http://localhost:8080/hello/");
         ws.Run();
         Console.WriteLine("Hello World");
         Console.ReadKey();
         ws.Stop();
      }
   }
}
```
As you can see, a port is open (8080). The C# webapp hello world application to response "Hello World" if triggered via this port and the /hello/ API.
To build this application, and to generate the executable binary, please issue the following command:
```
mcs helloweb.cs
```
As we did before, to use the executable binary, please issue the following command:
```
mono helloweb.exe
```



# Conclusions
