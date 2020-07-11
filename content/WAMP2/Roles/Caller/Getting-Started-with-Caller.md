+++
title = "Getting started with Caller"
categories = ["getting started", "caller"]
tags = ["caller"]
date = "2017-07-30T00:16:09+03:00"
+++

Before you start, see [Getting started with WAMPv2]({{< ref "WAMP2/Getting-started-with-WAMPv2.md" >}}) and create a WampChannel/WampHost your caller will use.

## About Caller role

WAMPv2 defines a Caller role, that is a role that can call a remote procedure call registered by a [Callee]({{< ref "WAMP2/Roles/Callee/Getting-Started-with-Callee.md" >}}) (using the CALL messages). The caller can receive a response with a result or an error from the router  (using the RESULT/ERROR message).

### Callee proxy

The Callee proxy is the easiest way to call rpc methods of a WAMP router.

In order to use it, create an interface having methods decorated with [WampProcedure] attribute.
Then create a proxy to a callee using the GetCalleeProxy method of the Services property  
of the IWampRealm/IWampRealmProxy instance.

#### Client sample

```csharp
using System;
using System.Threading.Tasks;
using WampSharp.V2;
using WampSharp.V2.Rpc;

namespace MyNamespace
{
    public interface IArgumentsService
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

    internal class Program
    {
        public static async Task Main(string[] args)
        {
            DefaultWampChannelFactory factory =
                new DefaultWampChannelFactory();

            const string serverAddress = "ws://127.0.0.1:8080/ws";

            IWampChannel channel =
                factory.CreateJsonChannel(serverAddress, "realm1");

            await channel.Open().ConfigureAwait(false);

            IArgumentsService proxy =
                channel.RealmProxy.Services.GetCalleeProxy<IArgumentsService>();

            await proxy.PingAsync().ConfigureAwait(false);
            Console.WriteLine("Pinged!");

            int result = await proxy.Add2Async(2, 3).ConfigureAwait(false);
            Console.WriteLine($"Add2: {result}");

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

            Console.ReadLine();
        }
    }
}
```

#### Router sample

```csharp
using System;
using System.Threading.Tasks;
using WampSharp.V2;
using WampSharp.V2.Realm;
using WampSharp.V2.Rpc;

namespace MyNamespace
{
    public interface IArgumentsService
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

    internal class Program
    {
        public static async Task Main(string[] args)
        {
            const string serverAddress = "ws://127.0.0.1:8080/ws";

            DefaultWampHost host = new DefaultWampHost(serverAddress);

            host.Open();

            Console.WriteLine("Press enter when a client finishes registering methods");
            Console.ReadLine();

            IWampHostedRealm realm = host.RealmContainer.GetRealmByName("realm1");

            IArgumentsService proxy =
                channel.RealmProxy.Services.GetCalleeProxy<IArgumentsService>();

            await proxy.PingAsync().ConfigureAwait(false);
            Console.WriteLine("Pinged!");

            int result = await proxy.Add2Async(2, 3).ConfigureAwait(false);
            Console.WriteLine($"Add2: {result}");

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

            Console.ReadLine();
        }
    }
}
```

### See also

* [Reflection based Caller]({{< ref "WAMP2/Roles/Caller/Reflection-based-Caller.md" >}})
* [Raw Caller]({{< ref "WAMP2/Roles/Caller/Raw-Caller.md" >}})
