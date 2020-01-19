---
title: "_index"
date: 2020-01-18T00:12:00+00:00
draft: false
---

This site contains documentation for the [WampSharp project](http://github.com/Code-Sharp/WampSharp).

## Introduction

[WAMP (The Web Application Messaging Protocol)](http://wamp.ws) is an open standard WebSocket subprotocol that provides two application messaging patterns in one unified protocol: Remote Procedure Calls + Publish & Subscribe.   

[WampSharp](http://github.com/Code-Sharp/WampSharp) is a .NET open source implementation of WAMP which allows you to write RPC services and Pub/Sub based applications in a convenient way.

## Roadmap

* [Roadmap]({{<ref "Roadmap.md">}})

## WAMPv2 Documentation

### Tutorials
* [Getting started with WAMPv2]({{< ref "WAMP2/Getting-started-with-WAMPv2.md" >}})
* [Getting started with Callee]({{<ref "WAMP2/Roles/Callee/Getting-Started-with-Callee.md">}})
* [Getting started with Caller]({{<ref "WAMP2/Roles/Caller/Getting-Started-with-Caller.md">}})
* [Getting started with Publisher]({{<ref "WAMP2/Roles/Publisher/Getting-Started-with-Publisher.md">}})
* [Getting started with Subscriber]({{<ref "WAMP2/Roles/Subscriber/Getting-Started-with-Subscriber.md">}})

### Roles

#### Callee
* [Reflection based Callee]({{<ref "WAMP2/Roles/Callee/Reflection-based-Callee.md">}})
* [Raw Callee]({{<ref "WAMP2/Roles/Callee/Raw-Callee.md">}})

#### Caller
* [Reflection based Caller]({{<ref "WAMP2/Roles/Caller/Reflection-based-Caller.md">}})
* [Raw Caller]({{<ref "WAMP2/Roles/Caller/Raw-Caller.md">}})

#### Publisher
* [Rx based Publisher]({{<ref "WAMP2/Roles/Publisher/Rx-based-Publisher.md">}})
* [Reflection based Publisher]({{<ref "WAMP2/Roles/Publisher/Reflection-based-Publisher.md">}})
* [Raw Publisher]({{<ref "WAMP2/Roles/Publisher/Raw-Publisher.md">}})

#### Subscriber
* [Rx based Subscriber]({{<ref "WAMP2/Roles/Subscriber/Rx-based-Subscriber.md">}})
* [Reflection based Subscriber]({{<ref "WAMP2/Roles/Subscriber/Reflection-based-Subscriber.md">}})
* [Raw Subscriber]({{<ref "WAMP2/Roles/Subscriber/Raw-Subscriber.md">}})

### Advanced profile features

The following [Advanced profile features](https://wamp-proto.org/static/rfc/draft-oberstet-hybi-crossbar-wamp.html#advanced-profile-1) are supported

* [Progressive call results](https://wamp-proto.org/static/rfc/draft-oberstet-hybi-crossbar-wamp.html#progressive-call-results): [caller tutorial]({{<ref "WAMP2/Roles/Caller/Reflection-based-Caller.md">}}#progressive-call-results), [callee tutorial]({{<ref "WAMP2/Roles/Callee/Reflection-based-Callee.md">}}#progressive-call-results)
* [Cancelation](https://wamp-proto.org/static/rfc/draft-oberstet-hybi-crossbar-wamp.html#call-canceling): [caller tutorial]({{<ref "WAMP2/Roles/Caller/Reflection-based-Caller.md">}}#cancellation), [callee tutorial]({{<ref "WAMP2/Roles/Callee/Reflection-based-Callee.md">}}#cancellation)
* [Caller identification](https://wamp-proto.org/static/rfc/draft-oberstet-hybi-crossbar-wamp.html#caller-identification): [caller tutorial]({{<ref "WAMP2/Roles/Caller/Reflection-based-Caller.md">}}#caller-identification), [callee tutorial]({{<ref "WAMP2/Roles/Callee/Reflection-based-Callee.md">}})
* [Session meta api](https://wamp-proto.org/static/rfc/draft-oberstet-hybi-crossbar-wamp.html#session-meta-api), [Registration meta api](https://wamp-proto.org/static/rfc/draft-oberstet-hybi-crossbar-wamp.html#registration-meta-api), [Subscription meta api](https://wamp-proto.org/static/rfc/draft-oberstet-hybi-crossbar-wamp.html#subscription-meta-api) - see [here]({{<ref "WAMP2/Meta-api-service.md">}})
* [Testament](https://wamp-proto.org/static/rfc/draft-oberstet-hybi-crossbar-wamp.html#testament) - see [here]({{<ref "WAMP2/Testament-service.md">}}).
* [Shared registrations](https://wamp-proto.org/static/rfc/draft-oberstet-hybi-crossbar-wamp.html#rfc.section.14.3.9), see also [here](http://crossbar.io/docs/Shared-Registrations/)  - see [callee tutorial]({{<ref "WAMP2/Roles/Callee/Reflection-based-Callee.md">}}#shared-registrations)
* [Subscriber black and whitelisting](https://wamp-proto.org/static/rfc/draft-oberstet-hybi-crossbar-wamp.html#subscriber-black-and-whitelisting)
* [Publisher exclusion](https://wamp-proto.org/static/rfc/draft-oberstet-hybi-crossbar-wamp.html#publisher-exclusion)
* [Publisher identification](https://wamp-proto.org/static/rfc/draft-oberstet-hybi-crossbar-wamp.html#publisher-identification)
* [Pattern-based subscriptions](https://wamp-proto.org/static/rfc/draft-oberstet-hybi-crossbar-wamp.html#pattern-based-subscriptions) - see also [here](http://crossbar.io/docs/Pattern-Based-Subscriptions/) - see [subscriber tutorial]({{<ref "WAMP2/Roles/Subscriber/Reflection-based-Subscriber.md">}}#pattern-based-subscriptions)
* [Pattern-based registrations](https://wamp-proto.org/static/rfc/draft-oberstet-hybi-crossbar-wamp.html#patternbased-registrations) - see also [here](http://crossbar.io/docs/Pattern-Based-Registrations/) - see [callee tutorial]({{<ref "WAMP2/Roles/Callee/Reflection-based-Callee.md">}}#pattern-based-registrations)
* [Event retention](https://github.com/wamp-proto/wamp-proto/blob/master/rfc/text/advanced/ap_pubsub_event_retention.md)
* [RawSocket transport](https://wamp-proto.org/static/rfc/draft-oberstet-hybi-crossbar-wamp.html#rawsocket) - see [here]({{<ref "WAMP2/Router/Transports/RawSocket.md">}})
* [Authentication](https://wamp-proto.org/static/rfc/draft-oberstet-hybi-crossbar-wamp.html#authentication) - see [Router side authentication]({{<ref "WAMP2/Router/Router-side-authentication.md">}}), [Client side authentication]({{<ref "WAMP2/Client/Client-side-authentication.md">}}).
* [WAMP-CRA](https://wamp-proto.org/static/rfc/draft-oberstet-hybi-crossbar-wamp.html#wampcra) - see [WAMP-CRA router side authentication]({{<ref "WAMP2/Router/WAMP-CRA-router-side-authentication.md">}}), [WAMP-CRA client side authentication]({{<ref "WAMP2/Client/WAMP-CRA-client-side-authentication.md">}})

### Router

* [WampHost]({{<ref "WAMP2/Router/WampHost.md">}})
* Authentication
  * [Router side authentication]({{<ref "WAMP2/Router/Router-side-authentication.md">}})
  * [WAMP-CRA router side authentication]({{<ref "WAMP2/Router/WAMP-CRA-router-side-authentication.md">}})
  * [Cookie based router side authentication]({{<ref "WAMP2/Router/Cookie-based-router-side-authentication.md">}})
* Transports
  * [WebSockets]({{<ref "WAMP2/Router/Transports/WebSocket.md">}})
      * [Fleck]({{<ref "WAMP2/Router/Transports/WebSocket.md">}}#wampsharp-fleck)
      * [Vtortola]({{<ref "WAMP2/Router/Transports/WebSocket.md">}}#wampsharp-vtortola)
      * [Http Listener]({{<ref "WAMP2/Router/Transports/WebSocket.md">}}#wampsharp-httplistener)
      * [ASP.NET Core]({{<ref "WAMP2/Router/Transports/WebSocket.md">}}#wampsharp-aspnetcore-websockets-server)
      * [ASP.NET]({{<ref "WAMP2/Router/Transports/WebSocket.md">}}#wampsharp-aspnet-websockets-server)
      * [Microsoft.Owin]({{<ref "WAMP2/Router/Transports/WebSocket.md">}}#wampsharp-owin)
  * [RawSocket]({{<ref "WAMP2/Router/Transports/RawSocket.md">}}#router-side-usage)
  * [SignalR](https://github.com/Code-Sharp/AutobahnJS.SignalR)

### Client

* [WampChannel]({{<ref "WAMP2/Client/WampChannel.md">}})
* [WampChannelReconnector]({{<ref "WAMP2/Client/WampChannelReconnector.md">}})
* Authentication
  * [Client side authentication]({{<ref "WAMP2/Client/Client-side-authentication.md">}})
  * [WAMP-CRA client side authentication]({{<ref "WAMP2/Client/WAMP-CRA-client-side-authentication.md">}})
* Transports
  * WebSocket4Net
  * [RawSocket]({{<ref "WAMP2/Router/Transports/RawSocket.md">}}#client-side-usage)
  * System.Net.WebSockets

## WAMPv1

* [Get Started!]({{<ref "WAMP1/Getting-started-with-WAMPv1.md">}})
* [Getting Started with WAMP client]({{<ref "WAMP1/Getting-started-with-WAMPv1-client.md">}})
* [Server RPC Hosting]({{<ref "WAMP1/Server RPC hosting (WAMPv1).md">}})
* [Server PubSub Hosting]({{<ref "WAMP1/Server PubSub hosting (WAMPv1).md">}})
* [Notes for WAMPv1 users]({{<ref "WAMP1/Notes-for-WAMPv1-users.md">}})

## Release Notes

* [WampSharp v20.1.1 release notes]({{<ref "Release-notes/WampSharp-v20.1.1-release-notes.md">}})
* [WampSharp v18.6.1 release notes]({{<ref "Release-notes/WampSharp-v18.6.1-release-notes.md">}})
* [WampSharp v1.2.6.41-beta release notes]({{<ref "Release-notes/WampSharp-v1.2.6.41-beta-release-notes.md">}})
* [WampSharp v1.2.5.21-beta release notes]({{<ref "Release-notes/WampSharp-v1.2.5.21-beta-release-notes.md">}})
* [WampSharp v1.2.4.18-beta release notes]({{<ref "Release-notes/WampSharp-v1.2.4.18-beta-release-notes.md">}})
* [WampSharp v1.2.3.12-beta release notes]({{<ref "Release-notes/WampSharp-v1.2.3.12-beta-release-notes.md">}})
* [WampSharp v1.2.2.8-beta release notes]({{<ref "Release-notes/WampSharp-v1.2.2.8-beta-release-notes.md">}})
* [WampSharp v1.2.1.6-beta release notes]({{<ref "Release-notes/WampSharp-v1.2.1.6-beta-release-notes.md">}})
