+++
title = "Rx-based Subscriber"
categories = ["subscriber"]
tags = ["subscriber", "rx"]
date = "2017-07-30T00:16:09+03:00"
+++

**Rx-based Subscriber** allows you to subscribe to events of a topic of a WAMP router realm, using [Reactive Extensions](http://reactivex.io/) Observable api.

## Basic usage

```csharp
private static async Task Main()
{
    DefaultWampChannelFactory factory = new DefaultWampChannelFactory();

    const string serverAddress = "ws://127.0.0.1:8080/ws";

    IWampChannel channel =
        factory.CreateJsonChannel(serverAddress, "realm1");

    await channel.Open().ConfigureAwait(false);

    IWampRealmProxy realmProxy = channel.RealmProxy;

    IDisposable subscription =
        realmProxy.Services.GetSubject<int>("com.myapp.topic1")
                  .Subscribe(x =>
                             {
                                 Console.WriteLine($"Got Event: {x}");
                             });

    // This line is required in order to release the WebSocket thread, otherwise it will be blocked by the Console.ReadLine() line.
    await Task.Yield();

    Console.WriteLine("Press enter to unsubscribe");

    Console.ReadLine();
    
    subscription.Dispose();
}
```

## GetSubject overloads

GetSubject allows you to subscribe to a topic of a WAMP realm, given the type of the received events.

If a generic type is specified, only the first argument in the Arguments parameter of the EVENT message is deserialized.

To handle other parameters of the EVENT message, one can use the IWampEventValueTupleConverter or IWampSubject overloads. These are described in the following sections.

### Tuples support

The interface IWampEventValueTupleConverter is responsible for converting a IWampSerializedEvent instance to a specified tuple and a specified tuple instance to a IWampEvent.

Luckily enough, in order to implement this interface, it suffices to derive from WampEventValueTupleConverter<> and specify the desired tuple type. Nothing else needs to be done.

(This might seem a bit odd, but that's the best way I'm aware of for preserving ValueTuple element names after compilation)

Then, just pass an instance of your IWampEventValueTupleConverter to the overload of IWampRealmServiceProvider's GetSubject method, which receives the topic's uri and an instance of IWampEventValueTupleConverter, in order to receive a ISubject<> instance of your desired tuple type.

```csharp
public static async Task Main()
{
    DefaultWampChannelFactory factory = new DefaultWampChannelFactory();

    IWampChannel channel =
        factory.CreateJsonChannel("ws://localhost:8080/ws", "realm1");

    await channel.Open().ConfigureAwait(false);

    ISubject<(int, int)> topic1Subject =
        channel.RealmProxy.Services.GetSubject
            ("com.myapp.topic1",
             new MyPositionalTupleEventConverter());

    topic1Subject.Subscribe(value =>
                            {
                                (int number1, int number2) = value;
                                Console.WriteLine($">com.myapp.topic1: Got event: number1:{number1}, number2:{number2}");
                            });

    ISubject<(int number1, int number2, string c, ComplexContract d)> topic2Subject =
        channel.RealmProxy.Services.GetSubject
            ("com.myapp.topic2",
             new MyKeywordTupleEventConverter());

    topic2Subject.Subscribe(value =>
                            {
                                (int number1, int number2, string c, ComplexContract d) = value;
                                Console.WriteLine($">com.myapp.topic2: Got event: number1:{number1}, number2:{number2}, c:{c}, d:{d}");
                            });

    // This line is required in order to release the WebSocket thread, otherwise it will be blocked by the following Console.ReadLine() line.
    await Task.Yield();

    Console.ReadLine();
}

public class MyPositionalTupleEventConverter : WampEventValueTupleConverter<(int, int)>
{
}

public class MyKeywordTupleEventConverter : WampEventValueTupleConverter<(int number1, int number2, string c, ComplexContract d)>
{
}

public class ComplexContract
{
    [JsonProperty("counter")]
    public int Counter { get; set; }

    [JsonProperty("foo")]
    public int[] Foo { get; set; }

    public override string ToString()
    {
        return string.Format("counter: {0}, foo: [{1}]",
                             Counter,
                             string.Join(", ", Foo));
    }
}
```

This code can consume events published by the following Javascript code:

```javascript
var counter = 0;

setInterval(function () {
    var obj = {'counter': counter, 'foo': [1, 2, 3]};

    session.publish('com.myapp.topic1', [randint(0, 100), 23], {});
    session.publish('com.myapp.topic2', [], {number1: randint(0, 100), number2: 23, c: "Hello", d: obj});

    counter += 1;

    console.log("events published");
}, 1000);
```

> This example is based on [this](https://github.com/tavendo/AutobahnPython/tree/master/examples/twisted/wamp/pubsub/complex) AutobahnJS sample.

### IWampSubject

Specifying no generic type to the GetSubject method will a return a IWampSubject, that is a IObservable of a IWampSerializedEvent. IWampSerializedEvent is an interface representing an incoming WAMP EVENT message. It has properties of type ISerializedValue that can be deserialized.

```csharp
public static async Task Main(string[] args)
{
    DefaultWampChannelFactory factory = new DefaultWampChannelFactory();

    const string serverAddress = "ws://127.0.0.1:8080/ws";

    IWampChannel channel =
        factory.CreateJsonChannel(serverAddress, "realm1");

    await channel.Open().ConfigureAwait(false);

    IWampRealmProxy realmProxy = channel.RealmProxy;

    IWampSubject topic =
        realmProxy.Services.GetSubject("com.myapp.topic2");

    IDisposable disposable = topic.Subscribe(OnTopic2);

    // This line is required in order to release the WebSocket thread, otherwise it will be blocked by the following Console.ReadLine() line.
    await Task.Yield();

    Console.WriteLine("Press enter to unsubscribe");

    Console.ReadLine();

    disposable.Dispose();
}

private static void OnTopic2(IWampSerializedEvent serializedEvent)
{
    int[] arguments =
        serializedEvent.Arguments.Select(argument => argument.Deserialize<int>())
                       .ToArray();

    string c =
        serializedEvent.ArgumentsKeywords["c"].Deserialize<string>();

    ComplexContract d =
        serializedEvent.ArgumentsKeywords["d"].Deserialize<ComplexContract>();

    var deserializedArguments =
        new
        {
            arguments,
            argumentsKeywords = new
                                {
                                    c,
                                    d
                                }
        };

    Console.WriteLine("Got event: args: [{0}], kwargs: {{ {1} }}",
                      string.Join(", ", deserializedArguments.arguments),
                      deserializedArguments.argumentsKeywords);
}

public class ComplexContract
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
```

> This example is based on [this](https://github.com/tavendo/AutobahnPython/tree/master/examples/twisted/wamp/pubsub/complex) AutobahnJS sample.