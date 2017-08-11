---
title: "_index"
date: 2017-07-30T00:16:09+03:00
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

The following [Advanced profile features](https://github.com/wamp-proto/wamp-proto/blob/master/rfc/text/advanced/ap_features.md) are supported

* [Progressive call results](https://github.com/wamp-proto/wamp-proto/tree/master/rfc/text/advanced/ap_rpc_progressive_call_results.md"): [caller tutorial]({{<ref "WAMP2/Roles/Caller/Reflection-based-Caller.md">}}#progressive-call-results), [callee tutorial]({{<ref "WAMP2/Roles/Callee/Reflection-based-Callee.md">}}#progressive-call-results)
* [Caller identification](https://github.com/wamp-proto/wamp-proto/tree/master/rfc/text/advanced/ap_rpc_caller_identification.md): [caller tutorial]({{<ref "WAMP2/Roles/Caller/Reflection-based-Caller.md">}}#caller-identification), [callee tutorial]({{<ref "WAMP2/Roles/Callee/Reflection-based-Callee.md">}})
* [Session meta api](https://github.com/wamp-proto/wamp-proto/blob/master/rfc/text/advanced/ap_session_meta_api.md), [Registration meta api](https://github.com/wamp-proto/wamp-proto/blob/master/rfc/text/advanced/ap_rpc_registration_meta_api.md), [Subscription meta api](https://github.com/wamp-proto/wamp-proto/blob/master/rfc/text/advanced/ap_pubsub_subscription_meta_api.md) - see [here]({{<ref "WAMP2/Meta-api-service.md">}})
* [Shared registrations](https://github.com/wamp-proto/wamp-proto/blob/master/rfc/text/advanced/ap_rpc_shared_registration.md), see also [here](http://crossbar.io/docs/Shared-Registrations/)  - see [callee tutorial]({{<ref "WAMP2/Roles/Callee/Reflection-based-Callee.md">}}#shared-registrations)
* [Subscriber black and whitelisting](https://github.com/wamp-proto/wamp-proto/blob/master/rfc/text/advanced/ap_pubsub_subscriber_blackwhite_listing.md)
* [Publisher exclusion](https://github.com/wamp-proto/wamp-proto/blob/master/rfc/text/advanced/ap_pubsub_publisher_exclusion.md)
* [Publisher identification](https://github.com/wamp-proto/wamp-proto/blob/master/rfc/text/advanced/ap_pubsub_publisher_identification.md)
* [Pattern-based subscriptions](https://github.com/wamp-proto/wamp-proto/blob/master/rfc/text/advanced/ap_pubsub_pattern_based_subscription.md) - see also [here](http://crossbar.io/docs/Pattern-Based-Subscriptions/) - see [subscriber tutorial]({{<ref "WAMP2/Roles/Subscriber/Reflection-based-Subscriber.md">}}#pattern-based-subscriptions)
* [Pattern-based registrations](https://github.com/wamp-proto/wamp-proto/blob/master/rfc/text/advanced/ap_rpc_pattern_based_registration.md) - see also [here](http://crossbar.io/docs/Pattern-Based-Registrations/) - see [callee tutorial]({{<ref "WAMP2/Roles/Callee/Reflection-based-Callee.md">}}#pattern-based-registrations)
* [RawSocket transport](https://github.com/wamp-proto/wamp-proto/blob/master/rfc/text/advanced/ap_transport_rawsocket.md) - see [here]({{<ref "WAMP2/Router/Transports/RawSocket.md">}})
* [Authentication](https://github.com/wamp-proto/wamp-proto/blob/master/rfc/text/advanced/ap_authentication.md) - see [Router side authentication]({{<ref "WAMP2/Router/Router-side-authentication.md">}}), [Client side authentication]({{<ref "WAMP2/Client/Client-side-authentication.md">}}).
* [WAMP-CRA](https://github.com/wamp-proto/wamp-proto/blob/master/rfc/text/advanced/ap_authentication_cra.md) - see [WAMP-CRA router side authentication]({{<ref "WAMP2/Router/WAMP-CRA-router-side-authentication.md">}}), [WAMP-CRA client side authentication]({{<ref "WAMP2/Client/WAMP-CRA-client-side-authentication.md">}})

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
      * [Vtortola]({{<ref "WAMP2/Router/Transports/WebSocket.md">}}#wampsharp-httplistener)
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

* [WampSharp v1.2.6.41-beta release notes]({{<ref "Release-notes/WampSharp-v1.2.6.41-beta-release-notes.md">}})
* [WampSharp v1.2.5.21-beta release notes]({{<ref "Release-notes/WampSharp-v1.2.5.21-beta-release-notes.md">}})
* [WampSharp v1.2.4.18-beta release notes]({{<ref "Release-notes/WampSharp-v1.2.4.18-beta-release-notes.md">}})
* [WampSharp v1.2.3.12-beta release notes]({{<ref "Release-notes/WampSharp-v1.2.3.12-beta-release-notes.md">}})
* [WampSharp v1.2.2.8-beta release notes]({{<ref "Release-notes/WampSharp-v1.2.2.8-beta-release-notes.md">}})
* [WampSharp v1.2.1.6-beta release notes]({{<ref "Release-notes/WampSharp-v1.2.1.6-beta-release-notes.md">}})
