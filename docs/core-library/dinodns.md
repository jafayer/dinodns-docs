---
title: DinoDNS
sidebar_position: 3
---

The `DinoDNS` class is intended to be a high-level, lightweight abstraction on top of the base [DefaultServer](./default-server/default-server.md) class. It accepts the same interface as `DefaultServer`, with the addition of optional parameters for each of the core plugin types: `cache`, `store`, and `logger`.

The `DinoDNS` class then takes care of registering the plugins in a sensible order, abstracting away some boilerplate that would otherwise be required to configure the server. Notably, it:

- Registers the logger handler as middleware
- Registers the cache handler as middleware
- Subscribes the cache to insert records when answers are returned to the client
- Registers the storage handler as middleware

Because `DinoDNS` extends `DefaultServer`, once you configure it you are still able to use all of the usual methods to add middleware, handlers, and to manage the state of the server.

```ts title="dinodns.ts"
const cache = new DefaultCache({
  maxEntries: 256,
});
const logger = new ConsoleLogger();
const store = new DefaultStore();

store.set('example.com', 'A', '127.0.0.1');
store.set('example.com', 'AAAA', '::1');
store.set('example.com', 'TXT', 'Hello, World!');

const server = new DinoDNS({
  cache,
  logger,
  storage: store,
  networks: [
    new DNSOverTCP({ address: '0.0.0.0', port: 1053 }),
    new DNSOverUDP({ address: '0.0.0.0', port: 1053 })
  ],
});
```
