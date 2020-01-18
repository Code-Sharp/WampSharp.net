+++
title = "RawSocket transport"
categories = ["router", "client"]
tags = ["router", "transports"]
date = "2017-08-01T10:25:07+09:00"
+++

WampSharp supports the [RawSocket transport](https://github.com/wamp-proto/wamp-proto/blob/master/rfc/text/advanced/ap_transport_rawsocket.md).
It is available through the [WampSharp.RawSocket](https://www.nuget.org/packages/WampSharp.RawSocket) NuGet package.

### Router-side usage

### TcpListener implementation

This is a standalone implementation which uses [`TcpListener`](https://docs.microsoft.com/en-us/dotnet/api/system.net.sockets.tcplistener). In order to use it, install the [WampSharp.RawSocket](https://www.nuget.org/packages/WampSharp.RawSocket) NuGet package. Usage example:

```csharp
IWampHost host = new WampHost();

RawSocketTransport transport =
    new RawSocketTransport(TcpListener.Create(8080));

host.RegisterTransport(transport,
                       new JTokenJsonBinding());

```

### ASP.NET Core implementation

This is an implementation of the RawSocket protocol using [Microsoft.AspNetCore.Connections.Abstractions](https://www.nuget.org/packages/Microsoft.AspNetCore.Connections.Abstractions/) package, which is part of [Project Bedrock](https://github.com/aspnet/KestrelHttpServer/issues/1980). This implementation is present in the package [WampSharp.AspNetCore.RawSocket](http://nuget.org/packages/WampSharp.AspNetCore.RawSocket).

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

### Client-side usage

In order to obtain a RawSocket IWampChannel, you can use the fluent syntax api:

```csharp
IWampChannelFactory factory = new WampChannelFactory();

IWampChannel channel =
    factory.ConnectToRealm("realm1")
           .RawSocketTransport("127.0.0.1", 8080)
           .MsgpackSerialization()
           .Build();

```

This fluent-api allows some RawSocket customization features:

```csharp
IWampChannelFactory factory = new WampChannelFactory();

IWampChannel channel =
    factory.ConnectToRealm("realm1")
           .RawSocketTransport("127.0.0.1", 8080)
           .ConnectFrom(new IPEndPoint(IPAddress.Loopback, 1345))
           // Chooses the port to connect from
           .AutoPing(TimeSpan.FromSeconds(45))
           // Enables auto-ping
           .MsgpackSerialization()
           .Build();
```

> Note: currently [crossbar](https://crossbar.io) and Autobahn variants [don't implement RawSocket's ping/pongs](https://github.com/crossbario/crossbar/issues/381). Therefore, auto-ping is disabled by default. You can enable it manually, both for router-side transport (by passing to the RawSocketTransport constructor an non-null auto-ping interval) and for client-side transport (as in the sample).