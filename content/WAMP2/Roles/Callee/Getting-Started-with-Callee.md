+++
title = "Getting started with Callee"
categories = ["getting started", "callee"]
tags = ["callee"]
date = "2017-07-30T00:16:09+03:00"
+++


Before you begin, see [Getting started with WAMPv2]({{< ref "WAMP2/Getting-started-with-WAMPv2.md" >}}) and create a WampChannel/WampHost your calleee will be registered to.


## About Callee role

WAMPv2 defines a Callee role, that is a role that can register a remote procedure call to the router (using the REGISTER/UNREGISTER messages). The callee's procedure can be invoked by the router (using the INVOCATION message). The callee can respond with a result or an error to the router  (using the YIELD/ERROR message).

## Reflection based callee

The Reflection based callee is the easiest way to register callee methods to a WAMP router.

In order to use it, create a class having methods decorated with [WampProcedure] attribute.
Then create an instance of the class and register it using the RegisterCalee method of the Services property of the IWampRealmProxy/IWampRealm instance.

### Client sample

```csharp
using System;
using System.Linq;
using System.Threading.Tasks;
using WampSharp.V2;
using WampSharp.V2.Client;
using WampSharp.V2.Rpc;

namespace MyNamespace
{
    public interface IArgumentsService
    {
        [WampProcedure("com.arguments.ping")]
        void Ping();

        [WampProcedure("com.arguments.add2")]
        int Add2(int a, int b);

        [WampProcedure("com.arguments.stars")]
        string Stars(string nick = "somebody", int stars = 0);

        [WampProcedure("com.arguments.orders")]
        string[] Orders(string product, int limit = 5);
    }

    public class ArgumentsService : IArgumentsService
    {
        public void Ping()
        {
        }

        public int Add2(int a, int b)
        {
            return a + b;
        }

        public string Stars(string nick = "somebody", int stars = 0)
        {
            return $"{nick} starred {stars}x";
        }

        public string[] Orders(string product, int limit = 5)
        {
            return Enumerable.Range(0, 50).Take(limit).Select(i => $"Product {i}").ToArray();
        }
    }

    internal class Program
    {
        public static async Task Main(string[] args)
        {
            const string location = "ws://127.0.0.1:8080/ws";

            DefaultWampChannelFactory channelFactory = new DefaultWampChannelFactory();

            IWampChannel channel = channelFactory.CreateJsonChannel(location, "realm1");

            Task openTask = channel.Open();

            await openTask.ConfigureAwait(false);

            IArgumentsService instance = new ArgumentsService();

            IWampRealmProxy realm = channel.RealmProxy;

            Task<IAsyncDisposable> registrationTask = realm.Services.RegisterCallee(instance);

            await registrationTask.ConfigureAwait(false);

            // This line is required in order to release the WebSocket thread, otherwise it will be blocked by the Console.ReadLine() line.
            await Task.Yield();

            Console.ReadLine();
        }
    }
}
```

### Router sample

```csharp
using System;
using System.Linq;
using System.Threading.Tasks;
using WampSharp.V2;
using WampSharp.V2.Realm;
using WampSharp.V2.Rpc;

namespace MyNamespace
{
    public interface IArgumentsService
    {
        [WampProcedure("com.arguments.ping")]
        void Ping();

        [WampProcedure("com.arguments.add2")]
        int Add2(int a, int b);

        [WampProcedure("com.arguments.stars")]
        string Stars(string nick = "somebody", int stars = 0);

        [WampProcedure("com.arguments.orders")]
        string[] Orders(string product, int limit = 5);
    }

    public class ArgumentsService : IArgumentsService
    {
        public void Ping()
        {
        }

        public int Add2(int a, int b)
        {
            return a + b;
        }

        public string Stars(string nick = "somebody", int stars = 0)
        {
            return $"{nick} starred {stars}x";
        }

        public string[] Orders(string product, int limit = 5)
        {
            return Enumerable.Range(0, 50).Take(limit).Select(i => $"Product {i}").ToArray();
        }
    }

    internal class Program
    {
        public static async Task Main(string[] args)
        {
            const string location = "ws://127.0.0.1:8080/ws";

            using (IWampHost host = new DefaultWampHost(location))
            {
                IArgumentsService instance = new ArgumentsService();

                IWampHostedRealm realm = host.RealmContainer.GetRealmByName("realm1");

                Task<IAsyncDisposable> registrationTask = realm.Services.RegisterCallee(instance);

                await registrationTask;

                host.Open();

                Console.WriteLine("Server is running on " + location);
                Console.ReadLine();
            }
        }
    }
}
```

### See also

* [Reflection based Callee]({{< ref "WAMP2/Roles/Callee/Reflection-based-Callee.md" >}})
* [Raw Callee]({{< ref "WAMP2/Roles/Callee/Raw-Callee.md" >}})
