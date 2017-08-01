+++
title = "Rx-based Publisher"
categories = ["publisher"]
tags = ["publisher", "rx"]
date = "2017-03-05T10:25:07+09:00"
+++

**Rx-based Publisher** allows you to publish events to a topic of a WAMP router realm, using [Reactive Extensions](http://reactivex.io/) Observer api.

## Basic usage

```csharp
private static void Run()
{
    const string serverAddress = "ws://127.0.0.1:8080/ws";

    DefaultWampChannelFactory channelFactory = new DefaultWampChannelFactory();

    IWampChannel channel = channelFactory.CreateJsonChannel(serverAddress, "realm1");

    Task openTask = channel.Open();

    openTask.Wait(5000);

    IWampRealmProxy realm = channel.RealmProxy;

    ISubject<int> subject =
        realm.Services.GetSubject<int>("com.myapp.topic1");

    // Publishes 5 to com.myapp.topic1
    subject.OnNext(5);
}
```

## GetSubject overloads

If a generic type is specified in the GetSubject method, the ARGUMENTS parameter of the EVENT message will contain the single parameter OnNext receives.

To send further arguments, one can use the IWampEventValueTupleConverter or IWampSubject overloads. These are described in the following sections.


### Tuples support

The interface IWampEventValueTupleConverter is responsible for converting a IWampSerializedEvent instance to a specified tuple and a specified tuple instance to a IWampEvent.

Luckily enough, in order to implement this interface, it suffices to derive from WampEventValueTupleConverter<> and specify the desired tuple type. Nothing else needs to be done.

(This might seem a bit odd, but that's the best way I'm aware of for preserving ValueTuple element names after compilation)

Then, just pass an instance of your IWampEventValueTupleConverter to the overload of IWampRealmServiceProvider's GetSubject method, which receives the topic's uri and an instance of IWampEventValueTupleConverter, in order to receive a ISubject<> instance of your desired tuple type.

```csharp
public async Task Run()
{
	DefaultWampChannelFactory factory = new DefaultWampChannelFactory();

	IWampChannel channel =
		factory.CreateJsonChannel("ws://localhost:8080/ws", "realm1");

	await channel.Open();

	ISubject<(int, int)> topic1Subject =
		channel.RealmProxy.Services.GetSubject
				("com.myapp.topic1",
				new MyPositionalTupleEventConverter());

	ISubject<(int number1, int number2, string c, ComplexContract d)> topic2Subject 
		channel.RealmProxy.Services.GetSubject
				("com.myapp.topic2",
				new MyKeywordTupleEventConverter());

	IObservable<int> timer =
		Observable.Timer(TimeSpan.FromMilliseconds(0),
						 TimeSpan.FromMilliseconds(1000))
						 .Select((x, i) => i);

	Random random = new Random();

	IDisposable disposable =
		timer.Subscribe(value =>
		{
			topic1Subject.OnNext((random.Next(0, 100), 23));
			topic2Subject.OnNext((random.Next(0, 100), 23, "Hello",
								  new ComplexContract()
								  {
									  Counter = value,
									  Foo = new int[] { 1, 2, 3 }
								  }));
		});
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

The following Javascript code can consume events published by the code above:

```javascript
function on_topic1(args, kwargs) {
    console.log("com.myapp.topic1: Got event:", args, kwargs);
}

function on_topic2(args, kwargs) {
    console.log("com.myapp.topic2: Got event:", args, kwargs);
}

session.subscribe('com.myapp.topic1', on_topic1);
session.subscribe('com.myapp.topic2', on_topic2);
```

> This example is based on [this](https://github.com/tavendo/AutobahnPython/tree/master/examples/twisted/wamp/pubsub/complex) AutobahnJS sample.

### IWampSubject

Specifying no generic type will a return a IWampSubject, that is a IObserver of a IWampEvent. IWampEvent is an interface representing an outgoing WAMP PUBLISH message. It has properties that represent PUBLISH message arguments.

```csharp
private static void Run()
{
    const string serverAddress = "ws://127.0.0.1:8080/ws";

    DefaultWampChannelFactory channelFactory = new DefaultWampChannelFactory();

    IWampChannel channel = channelFactory.CreateJsonChannel(serverAddress, "realm1");

    Task openTask = channel.Open();

    openTask.Wait(5000);

    IWampRealmProxy realm = channel.RealmProxy;

    IWampSubject subject =
        realm.Services.GetSubject("com.myapp.topic2");

    int counter = 0;

    IObservable<long> timer =
        Observable.Timer(TimeSpan.FromMilliseconds(0),
                         TimeSpan.FromMilliseconds(1000));

    Random random = new Random();

    IDisposable disposable =
        timer.Subscribe(x =>
        {
            var obj =
                new
                {
                    counter = counter,
                    foo = new int[] {1, 2, 3}
                };

            WampEvent @event = new WampEvent()
            {
                Options = new PublishOptions {DiscloseMe = true},
                Arguments = new object[] {random.Next(0, 100), 23},
                ArgumentsKeywords = new Dictionary<string, object>
                {
                    {"c", "Hello"},
                    {"d", obj}
                }
            };

            subject.OnNext(@event);
            Console.WriteLine("events published");

            counter += 1;
        });

    Console.ReadLine();
}
```

> This example is based on [this](https://github.com/tavendo/AutobahnPython/tree/master/examples/twisted/wamp/pubsub/complex) AutobahnJS sample.
