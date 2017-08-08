+++
title= "WampSharp v1.2.6.41-beta release notes"
categories = ["release notes"]
date= "2017-08-08T14:39:35+03:00"
+++

## New features

### New transports

A few router-side WebSockets transports have been added in this version. These include [HttpListener based transport]({{< ref "WAMP2\Router\Transports\WebSocket.md" >}}#wampsharp-httplistener), [ASP.NET based transport]({{< ref "WAMP2\Router\Transports\WebSocket.md" >}}#wampsharp-aspnet-websockets-server) and [Microsoft.Owin based transport]({{< ref "WAMP2\Router\Transports\WebSocket.md" >}}#wampsharp-owin).

### CANCEL support

CANCEL support has been added to [Crossbar](http://crossbar.io/) [recently](https://github.com/crossbario/crossbar/pull/1111). From this version, WampSharp supports rpc cancellations, both on router-side and on client-side.

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

> _Note_: This works also with progressive call results:
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

> _Note_: This works also with progressive call results:
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

> The code is based on [this sample](https://github.com/crossbario/crossbar-examples/tree/master/exclude_subscribers).