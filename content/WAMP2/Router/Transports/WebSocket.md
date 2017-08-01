+++
title = "WebSocket transports"
categories = ["router"]
tags = ["router", "transports"]
date = "2017-08-01T10:25:07+09:00"
+++

This pages describes the currently available router side WebSockets transports for WampSharp.

## WampSharp.Fleck

A [Fleck](https://github.com/statianzo/Fleck) based transport. It is the default transport for WampSharp (DefaultWampHost uses it). Supports .NET Framework 4.0.

Usage:

```csharp
WampHost host = new WampHost();

host.RegisterTransport(new FleckWebSocketTransport("ws://127.0.0.1:8080/ws"),
                       new JTokenJsonBinding(),
                       new JTokenMsgpackBinding());

host.Open();
```

Or

```csharp
WampHost host = new DefaultWampHost("ws://127.0.0.1:8080/ws");

host.Open();
```


## WampSharp.Vtortola

A transport based on [vtortola's WebSocketListener](https://github.com/vtortola/WebSocketListener). Supports [permessage-deflate](http://tools.ietf.org/html/draft-ietf-hybi-permessage-compression-17#section-8).

Usage:

```csharp
WampHost host = new WampHost();

IWampTransport transport =
    new VtortolaWebSocketTransport
        (endpoint: new IPEndPoint(IPAddress.Parse("127.0.0.1"), 8080),
            perMessageDeflate: true);

host.RegisterTransport(transport,
    new JTokenJsonBinding(),
    new JTokenMsgpackBinding());

host.Open();
```

## WampSharp.HttpListener

A transport based on [HttpListener](https://docs.microsoft.com/en-us/dotnet/framework/network-programming/httplistener).

Usage:

```csharp
const string serverAddress = "http://127.0.0.1:8080/ws/";

WampHost host = new WampHost();

host.RegisterTransport(new HttpListenerWebSocketTransport(serverAddress),
                       new JTokenJsonBinding());

host.Open();
```

## WampSharp.AspNetCore.WebSockets.Server

A transport for [ASP.NET Core](https://github.com/aspnet/Home). 

In order to use it, you need to create [a new empty ASP.NET Core project](https://docs.asp.net/en/latest/getting-started.html).

Install the following packages: WampSharp.AspNetCore.WebSockets.Server, WampSharp.NewtonsoftMsgpack (you can also install only WampSharp.NewtonsoftJson if you're not interested in MsgPack support).

Change your current Statup class to the following class

```csharp
public class Startup
{
    // This method gets called by the runtime. Use this method to add services to the container.
    // For more information on how to configure your application, visit http://go.microsoft.com/fwlink/?LinkID=398940
    public void ConfigureServices(IServiceCollection services)
    {
    }

    // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
    public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
    {
        loggerFactory.AddConsole();

        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }

        WampHost host = new WampHost();

        app.Map("/ws", builder =>
        {
            // Comment this out to test native server implementations
            builder.UseWebSockets(new WebSocketOptions
            {
                ReplaceFeature = true
            });

            host.RegisterTransport(new AspNetCoreWebSocketTransport(builder),
                                   new JTokenJsonBinding(),
                                   new JTokenMsgpackBinding());
        });

        host.Open();
    }
}
```

This code hosts a WampSharp Router in a ASP.NET Core application under the "/ws" path. The WampSharp relevant code begins in the line declaring the WampHost variable. ASP.NET Core runs by default from port 5000, so you'll need to access "ws://localhost:5000/ws". You can change the default port in various ways, such as calling `".UseUrls("http://127.0.0.1:8080")"` on the WebHostBuilder in your Main.

## WampSharp.AspNet.WebSockets.Server

A transport for [ASP.NET](https://www.asp.net/). 

In order to use it, add the following lines to your Global.asax.cs file.

```csharp
protected void Application_Start()
{
    WampHost host = new WampHost();

    // Listens at http://localhost:49875/ws where 49875 is the relevant port
    host.RegisterTransport(new AspNetWebSocketTransport("ws"),
                           new JTokenJsonBinding());

    host.Open();

    // ..
}
```

## Microsoft.Owin

A transport for [Microsoft.Owin](https://github.com/aspnet/AspNetKatana/).

Usage:

```csharp
using Owin;
using WampSharp.Binding;
using WampSharp.Owin;
using WampSharp.V2;

[assembly: OwinStartup(typeof(Startup))]

namespace WebApplication1
{
    public class Startup
    {
        public void Configuration(IAppBuilder app)
        {
            WampHost host = new WampHost();

            host.RegisterTransport(new OwinWebSocketTransport(app),
                                   new JTokenJsonBinding());

            host.Open();
        }
    }
}
```