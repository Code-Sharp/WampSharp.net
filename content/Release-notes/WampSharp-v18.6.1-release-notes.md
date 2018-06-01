---
title: "WampSharp v18.6.1 release notes"
date: 2018-06-01T09:51:52-04:00
categories: ["release notes"]
---

## New features

### ObserveOn extension method

From this version, the [client fluent syntax api]({{<ref "WAMP2/Client/WampChannel.md">}}#fluent-syntax) supports a new `ObserverOn` method. This method allows you to specify an [IScheduler](http://www.introtorx.com/Content/v1.0.10621.0/15_SchedulingAndThreading.html) that WAMP messages will be observed on. This for instance allows to ensure that [reflection based callee]({{<ref "WAMP2/Roles/Callee/Reflection-based-Callee.md">}}) methods or [reflection based subscriber]({{<ref "WAMP2/Roles/Subscriber/Reflection-based-Subscriber.md">}}) event handlers are invoked on the Dispatcher thread.

Usage example:

```csharp
IWampChannel channel =
    channelFactory.ConnectToRealm("realm1")
        .WebSocketTransport("ws://127.0.0.1:8080/ws")
        .JsonSerialization()
        .ObserveOn(DispatcherScheduler.Current)
        .Build();
```

### RawSocket Ssl support

From this version, RawSocket Ssl security is supported.

The RawSocket fluent api now includes a new extension method called `SslConfiguration`, which allows you to specify the arguments sent in the [AuthenticateAsClientAsync](https://docs.microsoft.com/en-us/dotnet/api/system.net.security.sslstream.authenticateasclientasync?view=netframework-4.7.1) method.

Similarly, the RawSocketTransport constructor now receives as a parameter a `ServerSslConfiguration` object, which allows you to specify the arguments sent in the [AuthenticateAsServerAsync](https://docs.microsoft.com/en-us/dotnet/api/system.net.security.sslstream.authenticateasserverasync?view=netframework-4.7.1) method.

### ASP.NET Core RawSocket implementation

This version includes an implementation of the RawSocket protocol using the new [Microsoft.AspNetCore.Connections.Abstractions](https://www.nuget.org/packages/Microsoft.AspNetCore.Connections.Abstractions/) package, part of [Project Bedrock](https://github.com/aspnet/KestrelHttpServer/issues/1980). This implementation is present in the package [WampSharp.AspNetCore.RawSocket](http://nuget.org/packages/WampSharp.AspNetCore.RawSocket).

Usage sample:

```csharp
using System.Net;
using Microsoft.AspNetCore;
using Microsoft.AspNetCore.Hosting;
using WampSharp.AspNetCore.RawSocket;
using WampSharp.Binding;
using WampSharp.V2;

public class Program
{
    public static void Main(string[] args)
    {
        WampHost wampHost = new WampHost();

        JTokenJsonBinding jsonBinding = new JTokenJsonBinding();

        IWebHost host =
            WebHost.CreateDefaultBuilder(args)
                   .UseKestrel(options =>
                               {
                                   options.Listen(IPAddress.Loopback, 8080,
                                                  builder =>
                                                  {
                                                      // Configure RawSocket transport
                                                      wampHost
                                                          .RegisterTransport(new AspNetCoreRawSocketTransport(builder),
                                                                             jsonBinding);
                                                  });
                               })
                   .Configure(app =>
                              {
                                  wampHost.Open();
                              })
                   .Build();

        host.Run();
    }
}
```

This allows you to listen to both RawSocket and WebSocket requests on the same port. For example, the following code listens both to WAMP RawSocket requests on port 8080 and to WAMP WebSocket requests on `ws://localhost:8080/ws/` (using the [WampSharp.AspNetCore.WebSockets.Server](http://nuget.org/packages/WampSharp.AspNetCore.WebSockets.Server) package):

```csharp
using System.Net;
using Microsoft.AspNetCore;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using WampSharp.AspNetCore.RawSocket;
using WampSharp.AspNetCore.WebSockets.Server;
using WampSharp.Binding;
using WampSharp.V2;

public class Program
{
    public static void Main(string[] args)
    {
        WampHost wampHost = new WampHost();

        JTokenJsonBinding jsonBinding = new JTokenJsonBinding();

        IWebHost host =
            WebHost.CreateDefaultBuilder(args)
                   .UseKestrel(options =>
                               {
                                   options.Listen(IPAddress.Loopback, 8080,
                                                  builder =>
                                                  {
                                                      // Log all of the http bytes as they are sent and received
                                                      builder.UseConnectionLogging();

                                                      // Configure RawSocket transport
                                                      wampHost
                                                          .RegisterTransport(new AspNetCoreRawSocketTransport(builder),
                                                                             jsonBinding);
                                                  });
                               })
                   .Configure(app =>
                              {
                                  app.Map("/ws",
                                          builder =>
                                          {
                                              builder.UseWebSockets();

                                              // Configure WebSockets transport
                                              wampHost.RegisterTransport
                                                  (new AspNetCoreWebSocketTransport(builder),
                                                   jsonBinding);
                                          });

                                  wampHost.Open();
                              })
                   .Build();

        host.Run();
    }
}
```

## Internal changes

* WampSharp.RawSocket now uses internally [Microsoft.IO.RecyclableMemoryStream](https://github.com/Microsoft/Microsoft.IO.RecyclableMemoryStream) instead of [System.Buffers](https://www.nuget.org/packages/System.Buffers/), in order to manage MemoryStream lifetimes, as the previous implementation used System.Buffers incorrectly.
* Optimizations for router-side pub/sub publications were made for RawSocket implementation. These also affect (minorly) the WampSharp.WebSockets based WebSockets implementations (i.e. ASP.NET, ASP.NET Core, Owin, HttpListener).
* A lot of auto refactoring was done to leverage C# 6.0 and C# 7.0 features. These include expression bodied properties, `nameof` operator, readonly properties, string interpolation, pattern matching, var out, etc. This should not affect the implementation in any way.
* Most of the dependencies were updated to latest version. As a result
    * [WampSharp.Fleck](https://www.nuget.org/packages/WampSharp.Fleck) and [WampSharp.Default.Router](https://www.nuget.org/packages/WampSharp.Default.Router) are now available for .NET Standard 2.0. 
    * The [System.Reactive](https://www.nuget.org/packages/System.Reactive) package reference was updated to v4.0.0, which now consists of a single dll, instead of 5 dlls.
    * If you need support for a framework variant which is not supported by [.NET Standard 2.0](http://immo.landwerth.net/netstandard-versions/#), please specify it and explain why in [this thread](https://github.com/Code-Sharp/WampSharp/issues/252).