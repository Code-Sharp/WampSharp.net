+++
title = "Reflection-based Subscriber"
categories = ["subscriber"]
tags = ["subscriber", "reflection"]
date = "2017-07-30T00:16:09+03:00"
+++

**Reflection-based Subscriber** allows to use WAMPv2 subscriber features in a similar fashion as [Reflection based Callee]({{< ref "WAMP2/Roles/Callee/Reflection-based-Callee.md" >}}).

In order to use it, create a class with a method having a [WampTopic] attribute, Then call RegisterSubscriber of IWampRealmServiceProvider.

Both placing attributes on a class method and placing attributes on an interface implemented by the class are supported.

## Basic usage

```csharp
public class MyClass
{
    [JsonProperty("counter")]
    public int Counter { get; set; }

    [JsonProperty("foo")]
    public int[] Foo { get; set; }

    public override string ToString()
    {
        return $"counter: {Counter}, foo: [{string.Join(", ", Foo)}]";
    }
}

public interface IMySubscriber
{
    [WampTopic("com.myapp.heartbeat")]
    void OnHeartbeat();

    [WampTopic("com.myapp.topic2")]
    void OnTopic2(int number1, int number2, string c, MyClass d);
}

public class MySubscriber : IMySubscriber
{
    public void OnHeartbeat()
    {
        long publicationId = WampEventContext.Current.PublicationId;
        Console.WriteLine($"Got heartbeat (publication ID {publicationId})");
    }

    public void OnTopic2(int number1, int number2, string c, MyClass d)
    {
        Console.WriteLine($"Got event: number1:{number1}, number2:{number2}, c:{c}, d:{d}");
    }
}

public static async Task Main()
{
    DefaultWampChannelFactory factory = new DefaultWampChannelFactory();

    IWampChannel channel =
        factory.CreateJsonChannel("ws://localhost:8080/ws", "realm1");

    await channel.Open().ConfigureAwait(false);

    Task<IAsyncDisposable> subscriptionTask =
        channel.RealmProxy.Services.RegisterSubscriber(new MySubscriber());

    IAsyncDisposable asyncDisposable = await subscriptionTask.ConfigureAwait(false);

    // This line is required in order to release the WebSocket thread, otherwise it will be blocked by the following Console.ReadLine() line.
    await Task.Yield();

    Console.WriteLine("Press enter to unsubscribe");

    Console.ReadLine();

    await asyncDisposable.DisposeAsync().ConfigureAwait(false);

    Console.WriteLine("Unsubscribed!");
}
```

>Note:  This sample is based on [this](https://github.com/tavendo/AutobahnPython/tree/master/examples/twisted/wamp/pubsub/complex) AutobahnJS sample

## Supported features

### WampEventContext

You can use WampEventContext.Current in order to get details about the current received event:

```csharp
public class MySubscriber
{
    [WampTopic("com.myapp.topic1")]
    public void OnTopic1(int counter)
    {
        WampEventContext context = WampEventContext.Current;

        Console.WriteLine("Got event, publication ID {0}, publisher {1}: {2}",
            context.PublicationId,
            context.EventDetails.Publisher,
            counter);
    }
}
```

>Note:  The sample is based on [this](https://github.com/tavendo/AutobahnPython/tree/master/examples/twisted/wamp/pubsub/options) AutobahnJS sample.

### Subscription customization

The RegisterSubscriber method of IWampRealmServiceProvider has an overload that receives an "interceptor" instance. The "interceptor" allow customizing the subscription being performed.

This allows customizing the subscribe options and the topic uri sent upon SUBSCRIBE message.

The following sections discuss subscribe options modifications that can be used in order to leverage WAMP advanced profile features.

### Pattern based subscriptions

 [Pattern based subscriptions](http://crossbar.io/docs/Pattern-Based-Subscriptions/), for both router side and client side.

In order to use it, pass SubscribeOptions with Match = "exact"/"prefix"/"wildcard" depending on your criteria (these are also available in a static class called WampMatchPattern):

```csharp
public static async Task Main()
{
    DefaultWampChannelFactory channelFactory = new DefaultWampChannelFactory();

    IWampChannel channel =
        channelFactory.CreateJsonChannel("ws://127.0.0.1:8080/ws",
                                         "realm1");

    await channel.Open().ConfigureAwait(false);

    await channel.RealmProxy.Services.RegisterSubscriber(new Subscriber1())
                 .ConfigureAwait(false);

    await channel.RealmProxy.Services.RegisterSubscriber
                     (new Subscriber2(),
                      new SubscriberRegistrationInterceptor(new SubscribeOptions
                                                            {
                                                                Match = WampMatchPattern.Prefix
                                                            }))
                 .ConfigureAwait(false);

    await channel.RealmProxy.Services.RegisterSubscriber
                     (new Subscriber3(),
                      new SubscriberRegistrationInterceptor(new SubscribeOptions
                                                            {
                                                                Match = WampMatchPattern.Wildcard
                                                            }))
                 .ConfigureAwait(false);

    // This line is required in order to release the WebSocket thread, otherwise it will be blocked by the Console.ReadLine() line.
    await Task.Yield();

    Console.ReadLine();
}

public class Subscriber1
{
    [WampTopic("com.example.topic1")]
    public void Handler1(string message)
    {
        Console.WriteLine($"handler1: msg = '{message}', topic = '{WampEventContext.Current.EventDetails.Topic}'");
    }
}

public class Subscriber2
{
    [WampTopic("com.example")]
    public void Handler2(string message)
    {
        Console.WriteLine($"handler2: msg = '{message}', topic = '{WampEventContext.Current.EventDetails.Topic}'");
    }             
}

public class Subscriber3
{
    [WampTopic("com..topic1")]
    public void Handler3(string message)
    {
        Console.WriteLine($"handler3: msg = '{message}', topic = '{WampEventContext.Current.EventDetails.Topic}'");
    }             
}
```
> Note: this sample is based on [this](https://github.com/crossbario/crossbarexamples/tree/master/patternsubs) Autobahn sample
