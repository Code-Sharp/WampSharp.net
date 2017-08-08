+++
title= "WampSharp v1.2.6.41-beta release notes"
categories = ["release notes"]
date= "2017-08-08T14:39:35+03:00"
+++

## New features

### New transports

A few router-side WebSockets transports have been added in this version. These include [HttpListener based transport]({{< ref "WAMP2\Router\Transports\WebSocket.md" >}}#wampsharp-httplistener), [ASP.NET based transport]({{< ref "WAMP2\Router\Transports\WebSocket.md" >}}#wampsharp-aspnet-websockets-server) and [Microsoft.Owin based transport]({{< ref "WAMP2\Router\Transports\WebSocket.md" >}}#wampsharp-owin).

### CANCEL support

CANCEL support has been added to Crossbar [recently](https://github.com/crossbario/crossbar/pull/1111). From this version, WampSharp supports rpc cancellations, both on router-side and on client-side.

#### Reflection based caller/callee cancellation support

Cancellation is supported for reflection based caller and callee via the [CancellationToken api](https://msdn.microsoft.com/en-us/library/system.threading.cancellationtoken(v=vs.110).aspx). In order to declare a callee procedure which supports cancellation, declare an async method which receive as the last argument of the procedure a Cancellation token:

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