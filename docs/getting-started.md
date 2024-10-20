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

DinoDNS provides low-dependency defaults for many of the plugins. A simple DNS server can be described as such:

```ts
import { DinoDNS } from  "dinodns";
import { DefaultStore } from "dinodns/plugins/storage";
import { DefaultCache } from "dinodns/plugins/cache";
import { DefaultLogger } from "dinodns/plugins/logging";
import { DNSOverTCP, DNSOverUDP } from "dinodns/networks";

const store = new DefaultStore();
const cache = new DefaultCache();
const logger = new DefaultLogger();

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

Read more at the DinoDNS page.

## DNSServer

The DNSServer is the fastest way to get up and running if you have custom plugin logic you want to include in your DNS server.

```ts
import { DNSServer } from "dinodns";
import { DNSOverTCP, DNSOverUDP } from "dinodns/networks";

const server = new DNSServer({
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

Read more at the DNSServer page.
