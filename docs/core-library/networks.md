---
title: Networks
sidebar_position: 4
---

Networks are the interfaces handle serializing queries and dispatching responses. Networks are responsible both for creating a server on their underlying protocol, and serializing messages out of the DNS wire format for their protocol. As such, defining a network typically involves defining both the base Network class, as well as a serializer.

The only packaged networks that are included in DinoDNS are `DNSOverTCP` and `DNSOverUDP`. However, there is also a `DNSOverHTTP` plugin that can be installed which supports HTTP/S. This was done to reduce bundled dependencies, as the `DNSOverHTTP` class has external dependencies.

## Usage

Networks are passed into the server's `networks` parameter. You can pass in any number of networks, meaning if you want your server to listen on more than one port, you can simply create more than one network interface for the protocol:

```ts
new DefaultServer({
    networks: [new DNSOverTCP('localhost', 53), new DNSOverTCP('localhost', 1053)],
    ...
})
```