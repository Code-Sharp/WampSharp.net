+++
title = "Getting started with WAMPv1"
categories = ["wamp1"]
tags = ["wamp1"]
date = "2017-07-30T00:16:09+03:00"
+++

Create a new Console Application in Visual Studio.

Open Package Manager Console (Tools -> Library Package Manager -> Package Manager Console) and enter the command

> Install-Package WampSharp.Default -Pre

This will install the pre-release version of WampSharp.

In your Program file, add the following code:

```csharp
using System;
using WampSharp.V1;
using WampSharp.V1.Rpc;

namespace MyNamespace
{
    public interface ICalculator
    {
        [WampRpcMethod("http://example.com/simple/calc#add")]
        int Add(int x, int y);
    }

    internal class Calculator : ICalculator
    {
        public int Add(int x, int y)
        {
            return x + y;
        }
    }

    class Program
    {
        public static void Main(string[] args)
        {
            const string location = "ws://127.0.0.1:9000/";
            using (IWampHost host = new DefaultWampHost(location))
            {
                ICalculator instance = new Calculator();
                host.HostService(instance);

                host.Open();

                Console.WriteLine("Server is running on " + location);
                Console.ReadLine();
            }
        }
    }
}
```

Compile and run.

Congratulations! you've just hosted your first WampSharp service!

Try it! enter this [autobahnjs demo client](http://autobahn.ws/static/file/autobahnjs.html) with a modern browser (Firefox or Chrome for instance). Open the web developer console (CTRL+Shift+I in Firefox and Chrome) - press the "Call Procedure" button - this calls your Add method with the parameters (23, 7).

To test pub/sub open two tabs of the [autobahnjs demo client](http://autobahn.ws/static/file/autobahnjs.html) and open the web developer console in each tab. Now press "Publish Event" in one of the tabs. Switch to the other tab. You should see a received event in the web developer console.