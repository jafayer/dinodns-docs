---
title: Networks
sidebar_position: 4
---

Networks are the interfaces handle serializing queries and dispatching responses. Networks are responsible both for creating a server on their underlying protocol, and serializing messages out of the DNS wire format for their protocol. As such, defining a network typically involves defining both the base Network class, as well as a serializer.

The only packaged networks that are included in DinoDNS are `DNSOverTCP` and `DNSOverUDP`, and `DNSOverHTTP` which also provides DNS over HTTPS.

Networks are passed into the server's `networks` parameter. You can pass in any number of networks, meaning if you want your server to listen on more than one port on the same protocol, you can simply create more than one network interface for the protocol:

```ts
new DefaultServer({
    networks: [
        new DNSOverTCP({ address: 'localhost', port: 53 }),
        new DNSOverTCP({ address: 'localhost', port: 53 }),
    ],
    ...
})
```

### `DNSOverUDP`

The UDP network is very low-configuration and merely accept a port and address used to start up the server. If you'd like, you can also provide a custom `serializer`, though the default serializer should fit nearly all use cases.

#### Usage
```ts
new DNSOverUDP({ address: 'localhost', port: 1053 })
```

### `DNSOverTCP`

Using the TCP network is much like the UDP network, with the added behavior that supplying an `ssl` parameter converts the network into a TLS network:

#### Usage
```ts
const tcp = new DNSOverTCP({ address: 'localhost', port: 1053 });
const tls = new DNSOverTCP({
    address: 'localhost',
    port: 1053,
    ssl: { key: '...', cert: '...' }
});
console.log(tcp.networkType) // "TCP"
console.log(tls.networkType) // "TLS"
```
### `DNSOverHTTP`

Like the TCP network, supplying the `ssl` parameter converts the HTTP network into an HTTPS network.

#### Usage
```ts
const http = new DNSOverHTTP({ address: 'localhost', port: 1080 });
const https = new DNSOverHTTP({
    address: 'localhost',
    port: 1443,
    ssl: { key: '...', cert: '...' }
});
console.log(http.networkType) // "HTTP"
console.log(https.networkType) // "HTTPS"
```
