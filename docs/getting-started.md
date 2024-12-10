---
sidebar_position: 2
title: Getting Started
---

# Installation

You can install DinoDNS via npm:

```sh
npm install dinodns --save
```

There are also external plugins that are not shipped with the core library so as to reduce the number of bundled dependencies.

# Usage

## DinoDNS

DinoDNS provides built-in, low-dependency defaults for many of the plugins required to run a DNS server, such as logging, caching, and record storage. The `DinoDNS` class is a higher-level implementation that accepts these plugins and automatically configures them to work with the server. A simple DNS server using the `DinoDNS` class can be described as such:

```ts
import { DinoDNS } from  "dinodns";
import { DNSOverTCP, DNSOverUDP } from "dinodns/common";
import { DefaultStore } from "dinodns/plugins/storage";
import { DefaultCache } from "dinodns/plugins/cache";
import { ConsoleLogger } from "dinodns/plugins/logging";

const store = new DefaultStore();
const cache = new DefaultCache();
const logger = new ConsoleLogger();

const server = new DinoDNS({
    storage: store,
    cache: cache,
    logger: logger,
    networks: [
        new DNSOverTCP({address: "localhost", port: 1053}),
        new DNSOverUDP({address: "localhost", port: 1053}),
    ]
});
```

The `DinoDNS` class abstracts away the actions required to register the plugins and wire them up to work correctly inside the server. Read more at the [DinoDNS page](/core-library/dinodns).

## DefaultServer

The `DefaultServer` is a slightly lower-level way to interact with the DinoDNS API. You can still use plugins easily, but you'll have to register their handlers yourself.

```ts
import { DefaultServer, DNSOverTCP, DNSOverUDP } from "dinodns/common";
import { DefaultStore } from "dinodns/plugins/store";

const server = new DefaultServer({
    networks: [
        new DNSOverTCP({address: "localhost", port: 1053}),
        new DNSOverUDP({address: "localhost", port: 1053}),
    ]
});

const store = new DefaultStore();

server.use(store.handler);
```

Both servers support writing and registering handlers and middleware inline with the `handle` and `use` methods:

```ts
server.use((req, res, next) => {});
server.handle((req, res, next) => {});
```

Read more at the [DefaultServer page](/core-library/default-server).
