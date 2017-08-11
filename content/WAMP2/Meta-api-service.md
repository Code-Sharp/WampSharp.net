+++
title = "Meta-api descriptor service"
categories = ["client", "router"]
tags = ["client", "router", "services"]
date = "2017-07-30T00:16:09+03:00"
+++

WampSharp implements WAMP meta-api (i.e. [session meta api](https://github.com/wamp-proto/wamp-proto/blob/master/rfc/text/advanced/ap_session_meta_api.md), [registration meta api](https://github.com/wamp-proto/wamp-proto/blob/master/rfc/text/advanced/ap_rpc_registration_meta_api.md) and [subscription meta api](https://github.com/wamp-proto/wamp-proto/blob/master/rfc/text/advanced/ap_pubsub_subscription_meta_api.md)). It is possible both to consume WAMP meta-api from a WampSharp client, and to expose it from a WampSharp router.

## Exposing meta-api

In order to expose meta-api, you can call an extension method of IWampHostedRealm, named "HostMetaApiService". This method returns an IDisposable which you can dispose in order to unregister the meta-api service.

> Note: it is important to call HostMetaApiService before hosting any other components (callees/subscribers), since otherwise the meta-api service isn't be able to track components registered before it.

```csharp
DefaultWampHost host = new DefaultWampHost("ws://127.0.0.1:8080/ws");

IWampHostedRealm realm = host.RealmContainer.GetRealmByName("realm1");

IDisposable disposable = realm.HostMetaApiService();

host.Open();
```

## Consuming meta-api

In order to consume meta-api, you can use declare the WAMP meta-api contracts yourself and consume it with plain WampSharp client-side code. In order to save that amount of work, a client-side api is provided which allows you to consume the meta-api without having to write any contracts yourself. This is api is available via an extension method of IWampRealmProxy which is named "GetMetaApiServiceProxy".

```csharp
private static async Task Run()
{
    WampChannelFactory channelFactory = new WampChannelFactory();

    IWampChannel channel =
        channelFactory.ConnectToRealm("realm1")
                      .WebSocketTransport("ws://127.0.0.1:8080/ws")
                      .JsonSerialization()
                      .Build();

    await channel.Open().ConfigureAwait(false);

    WampMetaApiServiceProxy proxy = channel.RealmProxy.GetMetaApiServiceProxy();

    long sessionCount = await proxy.CountSessionsAsync();

    IAsyncDisposable onCreateDisposable =
        await proxy.SubscribeTo.Subscription.OnCreate(
            (id, details) =>
            {
                Console.WriteLine($"Subscription with id {id} created, topic uri {details.Uri}");
            })
            .ConfigureAwait(false);

    IAsyncDisposable onJoinDisposable =
        await proxy.SubscribeTo.Session.OnJoin(
            details =>
            {
                Console.WriteLine($"Session with id {details.Session} joined");
            })
            .ConfigureAwait(false);
}
```
