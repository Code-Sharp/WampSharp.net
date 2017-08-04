+++
title = "RawSocket transport"
categories = ["router", "client"]
tags = ["router", "transports"]
date = "2017-08-01T10:25:07+09:00"
+++

WampSharp supports the [RawSocket transport](https://github.com/wamp-proto/wamp-proto/blob/master/rfc/text/advanced/ap_transport_rawsocket.md).
It is available through the [WampSharp.RawSocket](https://www.nuget.org/packages/WampSharp.RawSocket) NuGet package.

### Router-side usage

```csharp
IWampHost host = new WampHost();

RawSocketTransport transport =
    new RawSocketTransport(TcpListener.Create(8080));

host.RegisterTransport(transport,
                       new JTokenJsonBinding());

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