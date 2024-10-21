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

`DinoDNS` provides built-in, low-dependency defaults for many of the plugins required to run a DNS server, such as logging, caching, and record storage. A simple DNS server using the `DinoDNS` class can be described as such:

```ts
import { DinoDNS } from  "dinodns";
import { DefaultStore } from "dinodns/plugins/storage";
import { DefaultCache } from "dinodns/plugins/cache";
import { ConsoleLogger } from "dinodns/plugins/logging";
import { DNSOverTCP, DNSOverUDP } from "dinodns/networks";

const store = new DefaultStore();
const cache = new DefaultCache();
const logger = new ConsoleLogger();

const server = new DinoDNS({
    storage: store,
    cache: cache,
    logger: logger,
    networks: [
        new DNSOverTCP("localhost", 1053),
        new DNSOverUDP("localhost", 1053),
    ]
});
```

The `DinoDNS` class abstracts away the actions required to register the plugins and wire them up to work correctly inside the server. Read more at the [DinoDNS page](/core-library/dinodns).

## DefaultServer

The `DefaultServer` is the fastest way to get up and running if you have custom plugin logic you want to include in your DNS server and don't especially have a need for any higher-level plugins.

```ts
import { DefaultServer } from "dinodns";
import { DNSOverTCP, DNSOverUDP } from "dinodns/networks";

const server = new DefaultServer({
    networks: [
        new DNSOverTCP("localhost", 1053),
        new DNSOverUDP("localhost", 1053),
    ]
});
```

From there, you can write and register plugins and middleware inline with the `handle` and `use` methods:

```ts
server.use((req, res, next) => {});
server.handle((req, res, next) => {});
```

Read more at the [DefaultServer page](/core-library/default-server).
