+++
title= "WampSharp v1.2.6.41-beta release notes"
categories = ["release notes"]
date= "2017-08-08T14:39:35+03:00"
+++

## New features

### New transports

A few router-side WebSockets transports have been added in this version. These include [HttpListener based transport]({{< ref "WAMP2\Router\Transports\WebSocket.md" >}}#wampsharp-httplistener), [ASP.NET based transport]({{< ref "WAMP2\Router\Transports\WebSocket.md" >}}#wampsharp-aspnet-websockets-server) and [Microsoft.Owin based transport]({{< ref "WAMP2\Router\Transports\WebSocket.md" >}}#wampsharp-owin).

### CANCEL/INTERRUPT support

CANCEL/INTERRUPT support has been added to [Crossbar](http://crossbar.io/) [recently](https://github.com/crossbario/crossbar/pull/1111). From this version, WampSharp supports rpc cancellations, both on router-side and on client-side.

#### Reflection based caller/callee cancellation support

Cancellation is supported for reflection based caller and callee via the [CancellationToken api](https://msdn.microsoft.com/en-us/library/system.threading.cancellationtoken(v=vs.110).aspx). In order to declare a callee procedure which supports cancellation, declare an async method which receives as its last argument a CancellationToken:

```csharp
public class CancellableOpService
{
    [WampProcedure("com.myapp.cancellableop")]
    public async Task<int> CancellableOp(int n, CancellationToken token)
    {
        for (int i = 0; i < n; i++)
        {
            if (token.IsCancellationRequested)
            {
                throw new WampRpcCanceledException($" {i * 100.0 / n}% of the work was done");
            }

            await Task.Delay(100, token);
        }

        return n;
    }
}
```

> **Note**: This works also with progressive call results:
```csharp
public class LongCancellableOpService
{
    [WampProgressiveResultProcedure]
    [WampProcedure("com.myapp.longop")]
    public async Task<int> LongCancellableOp(int n, IProgress<int> progress, CancellationToken token)
    {
        for (int i = 0; i < n; i++)
        {
            if (token.IsCancellationRequested)
            {
                throw new WampRpcCanceledException($" {i * 100.0 / n}% of the work was done");
            }

            progress.Report(i);
            await Task.Delay(100, token);
        }

        return n;
    }
}
```

In order to cancel a pending rpc invocation from a WAMP caller using the callee proxy api, declare an interface containing an async method receiving a CancellationToken as its last parameter:

```csharp
public interface ICancellableOpService
{
    [WampProcedure("com.myapp.cancellableop")]
    Task<int> CancellableOp(int n, CancellationToken token);
}
```

Then pass a CancellationToken to the method call, this is usually done using CancellationTokenSource. Call CancellationTokenSource.Cancel in order to cancel the invocation.

```csharp
ICancellableOpService proxy =
    channel.RealmProxy.Services.GetCalleeProxy<ICancellableOpService>();

CancellationTokenSource cancellationTokenSource = new CancellationTokenSource();

Task<int> invocationTask = proxy.CancellableOp(4096, cancellationTokenSource.Token);

// Cancel the operation
cancellationTokenSource.Cancel();

await invocationTask.ConfigureAwait(false);
```

> **Note**: This works also with progressive call results:
```csharp
public interface ILongCancellableOpService
{
    [WampProcedure("com.myapp.cancellableop")]
    Task<int> LongCancellableOp(int n, IProgress<int> progress, CancellationToken token);
}
```
> 
> Cancellation:
```csharp
ILongCancellableOpService proxy =
    channel.RealmProxy.Services.GetCalleeProxy<ILongCancellableOpService>();

CancellationTokenSource cancellationTokenSource = new CancellationTokenSource();

Task<int> invocationTask = proxy.LongCancellableOp(4096, new MyProgress(), cancellationTokenSource.Token);

// Cancel the operation
cancellationTokenSource.Cancel();

await invocationTask.ConfigureAwait(false);
```

### Subscriber authid/authrole based black/white-listing ([Issue #119](https://github.com/Code-Sharp/WampSharp/issues/119))

From this version, [subscriber authid/authrole based black/white-listing](http://wamp-proto.org/static/rfc/draft-oberstet-hybi-crossbar-wamp.html#rfc.section.14.4.1.3) is supported.

Usage example:

```csharp
// Alice
IWampTopicProxy aliceHeartBeat =
    aliceChannel.RealmProxy.TopicContainer.GetTopicByUri("public.heartbeat.alice");

aliceHeartBeat.Publish(new PublishOptions() { ExcludeMe = false, ExcludeAuthenticationIds = new string[] { "bob" } }, new object[] { "From C#" });

// Bob
IWampTopicProxy bobHeartBeat =
    bobChannel.RealmProxy.TopicContainer.GetTopicByUri("public.heartbeat.bob");

bobHeartBeat.Publish(new PublishOptions() { ExcludeAuthenticationRoles = new string[] { "beta" } }, new object[] { "From C#" });

// Carol
IWampTopicProxy carolHeartBeat =
    carolChannel.RealmProxy.TopicContainer.GetTopicByUri("public.heartbeat.carol");

carolHeartBeat.Publish(new PublishOptions(), new object[] { "From C#" });

// Dave
IWampTopicProxy daveHeartBeat =
    daveChannel.RealmProxy.TopicContainer.GetTopicByUri("public.heartbeat.dave");

daveHeartBeat.Publish(new PublishOptions() { ExcludeAuthenticationIds = new string[] { "alice", "bob" } }, new object[] { "From C#" });

// Erin
IWampTopicProxy erinHeartBeat =
    erinChannel.RealmProxy.TopicContainer.GetTopicByUri("public.heartbeat.erin");

erinHeartBeat.Publish(new PublishOptions() { ExcludeMe = false, EligibleAuthenticationIds = new string[] { "alice", "bob", "dave" } }, new object[] { "From C#" });
erinHeartBeat.Publish(new PublishOptions() { ExcludeMe = false, EligibleAuthenticationRoles = new string[] { "beta" } }, new object[] { "From C#" });
```

> The sample code is based on [this Crossbar example](https://github.com/crossbario/crossbar-examples/tree/db85ad1b7f517d5ab46bf239559fa9155e599817/exclude_subscribers).

### Testament service

From this version, the [Testament feature](http://wamp-proto.org/static/rfc/draft-oberstet-hybi-crossbar-wamp.html#rfc.section.14.4.11) is supported.

This feature allows a client to request the router to publish some predefined events when the client disconnects.

In order to enable it from router-side, call HostTestamentService extension method of the relevant realm (this is similiar to the [Router-side meta-api usage]({{< ref "WAMP2\Meta-api-service.md" >}}#exposing-meta-api)):

```csharp
DefaultWampHost host = new DefaultWampHost("ws://127.0.0.1:8080/ws");

IWampHostedRealm realm = host.RealmContainer.GetRealmByName("realm1");

IDisposable disposable = realm.HostTestamentService();

// Uncomment to unregister the Testament service. 
// disposable.Dispose(); 

host.Open();
```

In order to consume the Testament service, one can define its contract and consume it using the CalleeProxy method. Another option is to use the GetTestamentServiceProxy extension method of IWampRealmProxy which returns such a proxy:

```csharp
private static async Task Run()
{
    WampChannelFactory channelFactory = new WampChannelFactory();

    IWampChannel channel =
        channelFactory.ConnectToRealm("realm1")
                      .WebSocketTransport("ws://127.0.0.1:8080/ws")
                      .JsonSerialization()
                      .Build();

    long? sessionId = null;

    channel.RealmProxy.Monitor.ConnectionEstablished += (sender, args) => { sessionId = args.SessionId; }; 

    await channel.Open().ConfigureAwait(false);

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

### Event Retention support

From this version WampSharp supports [Event Retention](https://github.com/wamp-proto/wamp-proto/blob/da34d9bd833beeb6f9cc8bc89faf8138d710aa78/rfc/text/advanced/ap_pubsub_event_retention.md), both on router-side and on client-side. This feature allows new subscribers of a given topic to receive upon subscribe the lastest retained event published to the topic before they subscribed to it.

A subscriber can indicate that it is interested in getting the retained event by specifying `GetRetained = true` in the SubscribeOptions argument.  

A publisher can indicate that it is interested in retaining a publication by specifying `Retain = true` in the PublisherOptions argument.

An example for a subscription which requests getting a retained event (Using [Reflection-based Subscriber]({{< ref "WAMP2\Roles\Subscriber\Reflection-based-Subscriber.md" >}})):

```csharp
public class MySubscriber
{
    [WampTopic("com.example.topic1")]
    public void OnCounter(int counter)
    {
        EventDetails details = WampEventContext.Current.EventDetails;

        if (details.Retained == true)
        {
            Console.WriteLine("RETAINED event received! " + counter);
        }
        else
        {
            Console.WriteLine("regular event received " + counter);
        }
    }
}

private static async Task Run()
{
    WampChannelFactory channelFactory = new WampChannelFactory();

    IWampChannel channel =
        channelFactory.ConnectToRealm("realm1")
                      .WebSocketTransport("ws://127.0.0.1:8080/ws")
                      .JsonSerialization()
                      .Build();

    await channel.Open().ConfigureAwait(false);

    try
    {
        IAsyncDisposable disposable =
            await channel.RealmProxy.Services.RegisterSubscriber(new MySubscriber(),
                                                                 new SubscriberRegistrationInterceptor(
                                                                     new SubscribeOptions()
                                                                     {
                                                                         GetRetained = true
                                                                     }))
                         .ConfigureAwait(false);

        Console.WriteLine("subscribed successfully");
    }
    catch (Exception ex)
    {
        Console.WriteLine("failed to subscribed: " + ex);
        throw;
    }
}
```

An example for a publisher which publishes retained events (Using [Reflection-based Publisher]({{< ref "WAMP2\Roles\Publisher\Reflection-based-Publisher.md" >}})):

```csharp
public class MyPublisher
{
    public MyPublisher()
    {
        IObservable<long> timer =
            Observable.Timer(TimeSpan.FromMilliseconds(0),
                             TimeSpan.FromMilliseconds(2000));

        timer.Select((x, i) => i)
             .Subscribe(i => OnCounter(i));
    }

    [WampTopic("com.example.topic1")]
    public event Action<int> Counter;

    protected virtual void OnCounter(int obj)
    {
        Counter?.Invoke(obj);
    }
}

private static async Task Run()
{
    WampChannelFactory channelFactory = new WampChannelFactory();

    IWampChannel channel =
        channelFactory.ConnectToRealm("realm1")
                      .WebSocketTransport("ws://127.0.0.1:8080/ws")
                      .JsonSerialization()
                      .Build();

    await channel.Open().ConfigureAwait(false);

    channel.RealmProxy.Services.RegisterPublisher(new MyPublisher(),
        new PublisherRegistrationInterceptor(new PublishOptions(){Retain = true}));
}
```

> **Note**: These samples are based on this [Crossbar example](https://github.com/crossbario/crossbar-examples/tree/4d498d72ecc3d46a35f146c7c9244d5319cf7cb5/pubsub/retained).

> **Note**: These samples only demonstrate the reflection-based api, but this also works with other overloads receiving SubscribeOptions/PublishOptions


## Internal/Breaking changes

* In order to support CANCEL/INTERRUPT, `IWampRpcOperation`'s `Invoke` methods now return `IWampCancellableInvocation`. If you don't want to support cancellation, you can simply return `null` from these methods.
* The following properties of `InvocationDetails` have been removed: `AuthenticationRole`, `AuthenticationId`, `AuthenticationMethod`. Instead, the following properties are available: `CallerAuthenticationRole`, `CallerAuthenticationId`. Similarly, the following properties of `EventDetails` have been removed: `AuthenticationRole`, `AuthenticationId`, `AuthenticationMethod`. Instead, the following properties are available: `PublisherAuthenticationRole`, `PublisherAuthenticationId`.
This has been done in order to make WampSharp compatible with the [recent changes](https://github.com/wamp-proto/wamp-proto/issues/57#issuecomment-234710359) done in Crossbar and Autobahn variants.
* Publishing to a topic having no subscribers no longer throws an exception and no longer sends a publish error to the publisher if the latter has requested publication acknowledgement. This is also Crossbar's behavior regarding topics without subscribers. This change also resolves [Issue #192](https://github.com/Code-Sharp/WampSharp/issues/192) implicitly.
* [LibLog](https://github.com/damianh/LibLog/) has been modified with some performance optimizations (see Pull Requests [#140](https://github.com/damianh/LibLog/pull/140) [#141](https://github.com/damianh/LibLog/pull/141)).