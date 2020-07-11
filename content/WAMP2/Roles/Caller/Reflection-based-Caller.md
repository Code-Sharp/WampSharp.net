+++
title = "Reflection-based Caller"
categories = ["caller"]
tags = ["caller", "reflection"]
date = "2017-07-30T00:16:09+03:00"
+++

**Reflection-based Caller** (or **"Callee proxy"**) allows you to call callee methods of a WAMP realm, by declaring an interface with methods decorated with a [WampProcedure] attribute.

The interface must be public.

## Basic usage

```csharp
public interface IArgumentsService
{
    [WampProcedure("com.arguments.add2")]
    int Add2(int a, int b);
}

public static void Main()
{
    DefaultWampChannelFactory factory =
        new DefaultWampChannelFactory();

    const string serverAddress = "ws://127.0.0.1:8080/ws";

    IWampChannel channel =
        factory.CreateJsonChannel(serverAddress, "realm1");

    channel.Open().Wait(5000);

    IArgumentsService proxy =
        channel.RealmProxy.Services.GetCalleeProxy<IArgumentsService>();

    int five = proxy.Add2(2, 3);
}
```

## Supported features

The following features are supported:

### Async method support

A method returning a Task&lt;&gt; can be awaited. Example:

```csharp
public interface IArgumentsServiceProxy
{
    [WampProcedure("com.arguments.ping")]
    Task PingAsync();

    [WampProcedure("com.arguments.add2")]
    Task<int> Add2Async(int a, int b);

    [WampProcedure("com.arguments.stars")]
    Task<string> StarsAsync(string nick = "somebody", int stars = 0);

    [WampProcedure("com.arguments.orders")]
    Task<string[]> OrdersAsync(string product, int limit = 5);
}
```

Call example:

```csharp
public static async Task Main()
{
    DefaultWampChannelFactory factory = new DefaultWampChannelFactory();

    const string serverAddress = "ws://127.0.0.1:8080/ws";

    IWampChannel channel =
        factory.CreateJsonChannel(serverAddress, "realm1");

    await channel.Open().ConfigureAwait(false);

    IArgumentsServiceProxy proxy =
        channel.RealmProxy.Services.GetCalleeProxy<IArgumentsServiceProxy>();

    await proxy.PingAsync().ConfigureAwait(false);
    Console.WriteLine("Pinged!");

    int result = await proxy.Add2Async(2, 3).ConfigureAwait(false);
    Console.WriteLine("Add2: {0}", result);

    var starred = await proxy.StarsAsync().ConfigureAwait(false);
    Console.WriteLine($"Starred 1: {starred}");

    starred = await proxy.StarsAsync(nick: "Homer").ConfigureAwait(false);
    Console.WriteLine($"Starred 2: {starred}");

    starred = await proxy.StarsAsync(stars: 5).ConfigureAwait(false);
    Console.WriteLine($"Starred 3: {starred}");

    starred = await proxy.StarsAsync(nick: "Homer", stars: 5).ConfigureAwait(false);
    Console.WriteLine($"Starred 4: {starred}");

    string[] orders = await proxy.OrdersAsync("coffee").ConfigureAwait(false);
    Console.WriteLine($"Orders 1: {string.Join(", ", orders)}");

    orders = await proxy.OrdersAsync("coffee", limit: 10).ConfigureAwait(false);
    Console.WriteLine($"Orders 2: {string.Join(", ", orders)}");
}
```

>Note:  The sample is based on [this](https://github.com/tavendo/AutobahnPython/tree/master/examples/twisted/wamp/rpc/arguments) AutobahnJS sample

### Tuples support

Reflection-based callers support C# 7.0 tuple return values. You can simply declare a method returning a C# 7.0 tuple in your callee proxy interface. The tuple will be deserialized either from the arguments keywords or from the arguments array of the RESULT message, depending on whether the returned tuple has named elements or positional elements. (tuples having elements which are partially named are not supported)

For example, declare the following callee proxy interface:

```csharp
public interface IComplexResultServiceProxy
{
	[WampProcedure("com.myapp.add_complex")]
	Task<(int c, int ci)> AddComplexAsync(int a, int ai, int b, int bi);

	[WampProcedure("com.myapp.split_name")]
    Task<(string, string)> SplitNameAsync(string fullname);

    [WampProcedure("com.myapp.add_complex")]
    (int c, int ci) AddComplex(int a, int ai, int b, int bi);

    [WampProcedure("com.myapp.split_name")]
    (string, string) SplitName(string fullname);
}
```

And then obtain a callee proxy and simply call its methods:

```csharp
public static async Task Main()
{
    DefaultWampChannelFactory factory = new DefaultWampChannelFactory();

    IWampChannel channel =
        factory.CreateJsonChannel("ws://localhost:8080/ws", "realm1");

    await channel.Open().ConfigureAwait(false);

    IComplexResultServiceProxy proxy =
        channel.RealmProxy.Services.GetCalleeProxy<IComplexResultServiceProxy>();

    (string forename, string surname) = await proxy.SplitNameAsync("Homer Simpson").ConfigureAwait(false);
    // Synchronous version: 
    // (string forename, string surname) = proxy.SplitName("Homer Simpson");

    Console.WriteLine($"Forename: {forename}, Surname: {surname}");

    (int c, int ci) = await proxy.AddComplexAsync(2, 3, 4, 5).ConfigureAwait(false);
    // Synchronous version: 
    // (int c, int ci) = proxy.AddComplex(2, 3, 4, 5);

    Console.WriteLine($"Result: {c} + {ci}i");
}
```

This code can consume the following code written in Javascript:

```javascript
function add_complex(args, kwargs) {
    return new autobahn.Result([], {c: args[0] + args[2], ci: args[1] + args[3]});
}

function split_name(args) {
    var splitted = args[0].split(" ");
    var forename = splitted[0];
    var surname = splitted[1];
    return new autobahn.Result([forename, surname]);
}

session.register('com.myapp.add_complex', add_complex).then(
    function (registration) {
        console.log("Procedure registered:", registration.id);
    },
    function (error) {
        console.log("Registration failed:", error);
    }
);

session.register('com.myapp.split_name', split_name).then(
    function (registration) {
        console.log("Procedure registered:", registration.id);
    },
    function (error) {
        console.log("Registration failed:", error);
    }
);
```

>Note:  The samples are based on [this](https://github.com/tavendo/AutobahnPython/tree/master/examples/twisted/wamp/rpc/complex) AutobahnJS/AutobahnPython sample

### Exception support
You can catch a WampException in order to treat a ERROR message.

Example:

```csharp
try
{
    await proxy.CheckNameAsync("Moses Montefiore").ConfigureAwait(false);
}
catch (WampException ex)
{
    string errorUri = ex.ErrorUri; // "com.myapp.error.invalid_length"
    IDictionary<string, object> arguments = ex.ArgumentsKeywords; // {"min": 3, "max": 10}
}
```
>Note:  The sample is based on [this](https://github.com/tavendo/AutobahnPython/tree/master/examples/twisted/wamp/rpc/errors) AutobahnJS sample

### Progressive call results

In order to use progressive call results as a Caller, declare in your callee service a [WampProcedure] method having a [WampProgressiveResultProcedure] attribute and a IProgress&lt;T&gt; as the last parameter.

> Note that the method return type should be Task&lt;T&gt; where this is the same T as in the IProgress&lt;T&gt; of the last parameter.

Example:

```csharp
public interface ILongOpService
{
    [WampProcedure("com.myapp.longop")]
    [WampProgressiveResultProcedure]
    Task<int> LongOp(int n, IProgress<int> progress);
}
```

Then obtain the proxy and call it:

```csharp
public static async Task Main()
{
    DefaultWampChannelFactory factory = new DefaultWampChannelFactory();

    IWampChannel channel = factory.CreateJsonChannel("ws://localhost:8080/ws", "realm1");

    await channel.Open()ConfigureAwait(false);

    ILongOpService proxy = channel.RealmProxy.Services.GetCalleeProxy<ILongOpService>();

    Progress<int> progress =
        new Progress<int>(i => Console.WriteLine("Got progress " + i));

    int result = await proxy.LongOp(10, progress).ConfigureAwait(false);

    Console.WriteLine("Got result " + result);
}
```

>Note:  The sample is based on [this](https://github.com/tavendo/AutobahnPython/tree/master/examples/twisted/wamp/rpc/progress) AutobahnJS sample

### Cancellation support

Cancellation is supported via the [CancellationToken api](https://msdn.microsoft.com/en-us/library/system.threading.cancellationtoken(v=vs.110).aspx). In order to cancel a pending rpc invocation from a WAMP caller using the callee proxy api, declare an interface containing an async method, receiving a CancellationToken as its last parameter:

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

### out/ref parameters

For synchronous methods, out/ref parameters are supported.

> Note: this is not supported for asynchronous methods.

Example:

```csharp
public interface IComplexResultService
{
    [WampProcedure("com.myapp.add_complex")]
    void AddComplex(int a, int ai, int b, int bi, out int c, out int ci);
}
```
Call example:

```csharp
int ci;
int c;
proxy.AddComplex(2, 3, 4, 5, out c, out  ci);
```
>Note:  The sample is based on [this](https://github.com/tavendo/AutobahnPython/tree/master/examples/twisted/wamp/rpc/complex) AutobahnJS sample

### Multi-valued results

In order to get an multivalued array from the RESULT/YIELD WAMPv2 message, set the return value of the rpc method to an array and place above it a [return: WampResult(CollectionResultTreatment.Multivalued)] attribute. Example:

```csharp
public interface IMultivaluedResultService
{
    [WampProcedure("com.myapp.split_name")]
    [return: WampResult(CollectionResultTreatment.Multivalued)]
    Task<string[]> SplitNameAsync(string fullname);
}
```

Call example:

```csharp
string[] splitted = proxy.SplitNameAsync("Homer Simpson");
```
>Note:  The sample is based on [this](https://github.com/tavendo/AutobahnPython/tree/master/examples/twisted/wamp/rpc/complex) AutobahnJS sample

## Call customization

The GetCalleeProxy method of IWampRealmServiceProvider now has an overload that receives an "interceptor" instance. The "interceptor" allows customizing the call being performed.

For instance, assume you want to call procedures of a contract that its procedures uris are known only on runtime.  This is possible implementing a ICalleeProxyInterceptor:

```csharp
public class MyCalleeProxyInterceptor : CalleeProxyInterceptor
{
    private readonly int mCalleeIndex;

    public MyCalleeProxyInterceptor(int calleeIndex) :
        base(new CallOptions())
    {
        mCalleeIndex = calleeIndex;
    }

    public override string GetProcedureUri(MethodInfo method)
    {
        string format = base.GetProcedureUri(method);
        string result = string.Format(format, mCalleeIndex);
        return result;
    }
}

```

This interceptor modifies the procedure uri of the procedure to call. For example, we can declare an interface with a method with this signature:

```csharp
public interface ISquareService
{
    [WampProcedure("com.myapp.square.{0}")]
    Task<int> Square(int number);
}
```

And then specify the index in runtime:

```csharp
public static async Task Main()
{
    DefaultWampChannelFactory factory = new DefaultWampChannelFactory();

    IWampChannel channel =
        factory.CreateJsonChannel("ws://localhost:8080/ws", "realm1");

    await channel.Open().ConfigureAwait(false);

    int index = GetRuntimeIndex();

    ISquareService proxy =
        channel.RealmProxy.Services.GetCalleeProxy<ISquareService>
        (new CachedCalleeProxyInterceptor(
            new MyCalleeProxyInterceptor(index)));

    int nine = await proxy.Square(3).ConfigureAwait(false); // Calls ("com.myapp.square." + index)
}
```

> Note: we wrap our interceptor with the CachedCalleeProxyInterceptor in order to cache the results of our interceptor, in order to avoid calculating them each call.

In addition, the interceptor allows to modify the options sent to each call. The following sections demonstrates modifications that can be used to leverage WAMP advanced profile features.

> Note: these interceptors are still "static", i.e: they don't allow returning a value that depends on the call parameters.

### Caller identification

Callee proxy sample:

According to WAMP2 specification, a Caller can request to disclose its identification (by specifying disclose_me = true on call request).

Specifying this is possible when obtaining callee proxy.

```csharp
public static async Task Main()
{
    DefaultWampChannelFactory factory = new DefaultWampChannelFactory();

    IWampChannel channel =
        factory.CreateJsonChannel("ws://localhost:8080/ws", "realm1");

    await channel.Open().ConfigureAwait(false);

    var callOptions = new CallOptions()
    {
        DiscloseMe = true
    };

    ISquareService proxy =
        channel.RealmProxy.Services.GetCalleeProxy<ISquareService>
        (new CachedCalleeProxyInterceptor(new CalleeProxyInterceptor(callOptions)));

    await proxy.Square(-2).ConfigureAwait(false);
    await proxy.Square(0).ConfigureAwait(false);
    await proxy.Square(2).ConfigureAwait(false);
}
```

> Note: The sample is based on [this](https://github.com/tavendo/AutobahnPython/tree/master/examples/twisted/wamp/rpc/options) AutobahnJS sample
