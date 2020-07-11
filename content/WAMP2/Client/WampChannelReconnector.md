+++
title = "WampChannelReconnector"
categories = ["client"]
tags = ["client"]
date = "2017-07-30T00:16:09+03:00"
+++

The WampChannelReconnector class helps handling re-connection of WampChannel.

Usage sample:

```csharp
public static async Task Main()
{
    DefaultWampChannelFactory factory = new DefaultWampChannelFactory();

    string address = "ws://localhost:8080/ws";

    MySubscriber mySubscriber = new MySubscriber();

    IWampChannel channel =
        factory.CreateJsonChannel(address, "realm1");

    Func<Task> connect = async () =>
    {
        await channel.Open().ConfigureAwait(false);

        var subscriptionTask =
            channel.RealmProxy.Services.RegisterSubscriber(mySubscriber);

        var asyncDisposable = await subscriptionTask.ConfigureAwait(false);
    };

    WampChannelReconnector reconnector =
        new WampChannelReconnector(channel, connect);

    reconnector.Start();

    // This line is required in order to release the WebSocket thread, otherwise it will be blocked by the following Console.ReadLine() line.
    await Task.Yield();

    Console.ReadLine();
}
```