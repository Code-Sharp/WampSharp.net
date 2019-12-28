---
title: "WampSharp v20.1.1 release notes"
date: 2019-12-28T11:38:22+02:00
categories: ["release notes"]
---

## Breaking changes

### Supported frameworks

From this version, WampSharp targets only the following .NET variants: [net461](https://github.com/microsoft/dotnet/tree/master/releases/net461), [netstandard2.0](https://devblogs.microsoft.com/dotnet/announcing-net-standard-2-0/), [netstandard2.1](https://devblogs.microsoft.com/dotnet/announcing-net-standard-2-1/). Support for the following variants has been dropped: `net40`, `net45`, `netstandard1.3`, `pcl`, `uap`.

.NET Standard 2.0/2.1 is the future for .NET libraries and every major .NET variant will implement .NET Standard. For instance, [UWP implements .NET Standard 2.0](https://devblogs.microsoft.com/dotnet/announcing-uwp-support-for-net-standard-2-0/).

### Removed packages

* The package `WampSharp.Windows` is removed - UWP targets should now use `WampSharp.WebSockets`.
* The package `WampSharp.SignalR` is removed.

### WampSharp.Default

* `DefaultWampChannelFactory` now uses `WampSharp.WebSockets` instead of `WampSharp.WebSocket4Net`. One can use `WampSharp.WebSocket4Net` instead by installing `WampSharp.WebSocket4Net` manually and calling the fluent syntax api `WebSocket4NetTransport` method.
* `DefaultWampChannelFactory` now uses `WampSharp.NewtonsoftMessagePack` instead of `WampSharp.NewtonsoftMsgpack`. One can use `WampSharp.NewtonsoftMsgpack` instead by installing `WampSharp.NewtonsoftMsgpack` manually and calling the fluent syntax api `MsgpackSerialization` method.
* `DefaultWampHost` now uses `WampSharp.NewtonsoftMessagePack` instead of `WampSharp.NewtonsoftMsgpack`. One can use `WampSharp.NewtonsoftMsgpack` instead by installing `WampSharp.NewtonsoftMsgpack` manually and creating a `DefaultWampHost` and calling its constructor with a list containing a `JTokenMsgpackBinding`.

### Proxy implementation

WampSharp had several implementations for its [Reflection based Caller]({{<ref "WAMP2/Roles/Caller/Reflection-based-Caller.md">}}) feature. For several platforms (`net40`, `net45`, `netstandard1.3`, `netstandard2.0`) it used [Castle.Core DynamicProxy](http://www.castleproject.org/projects/dynamicproxy/) and for others it used [System.Reflection.DispatchProxy](https://docs.microsoft.com/en-us/dotnet/api/system.reflection.dispatchproxy?view=netcore-3.1). From this version, `System.Reflection.DispatchProxy` is used for every platform. This is an internal change of the library that should not library users.

### IAsyncDisposable

In C# 8.0, the interface `IAsyncDisposable` [was introduced](https://docs.microsoft.com/en-us/dotnet/api/system.iasyncdisposable?view=netcore-3.1). Previous versions of WampSharp contained a similar interface. From this version, WampSharp uses the above `IAsyncDisposable` interface. This interface is only available in .NET Standard 2.1. For other variants, the interface is imported from the NuGet package [Microsoft.Bcl.AsyncInterfaces](https://www.nuget.org/packages/Microsoft.Bcl.AsyncInterfaces/).

Note that although the interfaces are similar, the signature of `DisposeAsync` is slightly different, as the new signature returns a `ValueTask`, while the previous signature returns a `Task`.

## Features

### Session management api

From this version, [WAMP session management api](https://wamp-proto.org/_static/gen/wamp_latest.html#x14-5-1-3-session-meta-procedures) is supported. This allows to disconnect router peers by calling rpc methods.

To host the session management service call the `HostSessionManagementService` extension method on your hosted realm object

```csharp
DefaultWampHost host = new DefaultWampHost("ws://127.0.0.1:8080/ws");

IWampHostedRealm realm = host.RealmContainer.GetRealmByName("realm1");

IDisposable disposable = realm.HostSessionManagementService();

// Uncomment to unregister the session management service.
// disposable.Dispose();

host.Open();
```

To disconnect a remote peer, obtain a proxy of and call one of its methods:

```csharp
WampChannelFactory channelFactory = new WampChannelFactory();

IWampChannel channel =
    channelFactory.ConnectToRealm("realm1")
                  .WebSocketTransport(new Uri("ws://127.0.0.1:8080/ws"))
                  .JsonSerialization()
                  .Build();

await channel.Open().ConfigureAwait(false);

IWampSessionManagementServiceProxy proxy = 
    channel.RealmProxy.Services.GetCalleeProxy<IWampSessionManagementServiceProxy>();

await proxy.KillBySessionIdAsync(15068678494738).ConfigureAwait(false);

await proxy.KillByAuthIdAsync("peter").ConfigureAwait(false);

await proxy.KillByAuthRoleAsync("frontend").ConfigureAwait(false);

await proxy.KillAllAsync().ConfigureAwait(false);
```

It is recommended to combine this feature with [authentication]({{<ref "WAMP2/Router/Router-side-authentication.md">}}), to ensure that unauthorized users can not disconnect other users.

### CBOR Support

From this version, WampSharp supports [CBOR serializaion](https://github.com/wamp-proto/wamp-proto/issues/151). This is provided by the `WampSharp.NewtonsoftCbor` package.

Client side usage:

```csharp
WampChannelFactory channelFactory = new WampChannelFactory();

IWampChannel channel =
    channelFactory.ConnectToRealm("realm1")
                  .WebSocketTransport(new Uri("ws://127.0.0.1:8080/ws"))
                  .CborSerialization()
                  .Build();

await channel.Open().ConfigureAwait(false);
```

Router side usage:

```csharp
WampHost host = new WampHost();

host.RegisterTransport(new FleckWebSocketTransport("ws://127.0.0.1:8080/ws"),
                       new JTokenJsonBinding(),
                       new JTokenCborBinding());


// Alternatively use:
// DefaultWampHost host =
//     new DefaultWampHost("ws://127.0.0.1:8080/ws",
//                         new List<IWampBinding>
//                         {
//                             new JTokenJsonBinding(),
//                             new JTokenCborBinding()
//                         });

host.Open();
```