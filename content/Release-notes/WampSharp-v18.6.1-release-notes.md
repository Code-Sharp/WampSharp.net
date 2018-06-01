---
title: "WampSharp v18.6.1 release notes"
date: 2018-06-01T09:51:52-04:00
categories: ["release notes"]
---

## New features

### ObserveOn extension method

From this version, the [client fluent syntax api]({{<ref "WAMP2/Client/WampChannel.md">}}#fluent-syntax) supports a new `ObserverOn` method. This method allows you to request that WAMP messages will be observed on a specified [IScheduler](http://www.introtorx.com/Content/v1.0.10621.0/15_SchedulingAndThreading.html). This for instance allows to ensure that [reflection based callee]({{<ref "WAMP2/Roles/Callee/Reflection-based-Callee.md">}}) methods or [reflection based subscriber]({{<ref "WAMP2/Roles/Subscriber/Reflection-based-Subscriber.md">}}) event handlers are invoked on the Dispatcher thread.

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