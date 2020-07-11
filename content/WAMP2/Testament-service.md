+++
title = "Testament service"
categories = ["client", "router"]
tags = ["client", "router", "services"]
date = "2017-07-30T00:16:09+03:00"
+++

The [Testament feature](http://wamp-proto.org/static/rfc/draft-oberstet-hybi-crossbar-wamp.html#rfc.section.14.4.11) is supported.

This feature allows a client to request the router to publish some predefined events when the client disconnects.

## Enabling the testament service on the router-side

In order to enable it from router-side, call HostTestamentService extension method of the relevant realm (this is similiar to the [Router-side meta-api usage]({{< ref "WAMP2/Meta-api-service.md" >}}#exposing-meta-api)):

```csharp
DefaultWampHost host = new DefaultWampHost("ws://127.0.0.1:8080/ws");

IWampHostedRealm realm = host.RealmContainer.GetRealmByName("realm1");

IDisposable disposable = realm.HostTestamentService();

// Uncomment to unregister the Testament service. 
// disposable.Dispose(); 

host.Open();
```

## Consuming the testament service

In order to consume the Testament service, one can define its contract and consume it using the CalleeProxy method. Another option is to use the GetTestamentServiceProxy extension method of IWampRealmProxy which returns such a proxy:

```csharp
private static async Task Run()
{
    WampChannelFactory channelFactory = new WampChannelFactory();

    IWampChannel channel =
        channelFactory.ConnectToRealm("realm1")
                      .WebSocketTransport(new Uri("ws://127.0.0.1:8080/ws"))
                      .JsonSerialization()
                      .Build();

    TaskCompletionSource<long> sessionIdTask = new TaskCompletionSource<long>();

    channel.RealmProxy.Monitor.ConnectionEstablished += (sender, args) => sessionIdTask.SetResult(args.SessionId);

    await channel.Open().ConfigureAwait(false);

    long sessionId = await sessionIdTask.Task.ConfigureAwait(false);

    IWampTestamentServiceProxy proxy = channel.RealmProxy.GetTestamentServiceProxy();

    string testamentTopic = "com.example.testament";

    for (int i = 0; i < 5; ++i)
    {
        object[] testamentArgs = new object[] {"this is my testament!"};

        Dictionary<string, object> testamentKwArgs =
            new Dictionary<string, object> {{"foo", i}, {"bar", "baz"}, {"sess", sessionId}};

        try
        {
            await proxy.AddTestamentAsync(testamentTopic, testamentArgs, testamentKwArgs)
                       .ConfigureAwait(false);

            Console.WriteLine("testament added");
        }
        catch (Exception ex)
        {
            Console.WriteLine("adding testament failed! " + ex);
        }
    }

    Console.WriteLine(@"Press any key to close channel. 5 events should be published to ""com.example.testament""");
    Console.ReadKey();
    channel.Close();
}
```

> This sample is based on this [Crossbar example](https://github.com/crossbario/crossbar-examples/tree/240c1f015990a859683a74de40e51a9f9ab3b5f9/pubsub/testament).
