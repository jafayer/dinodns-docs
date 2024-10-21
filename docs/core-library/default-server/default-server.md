---
title: DefaultServer
sidebar_position: 2
---

The `DefaultServer` is the main underlying class that's responsible for coordinating messages from the networks, constructing the plugin chain for incoming queries, and routing responses back to the client.

In its most basic form, the DefaultServer consists of just one or more networks and a simple plugin chain, as depicted below:

![A simple diagram of a DefaultServer](default-server.png)

## Usage

The `DefaultServer`'s only required parameter is an array of networks. However, you can also optionally provide a `defaultHandler` which will be executed if the server reaches the end of the plugin chain and no response has been generated, as well as a custom `Router` if the default router doesn't suit your needs.

```ts
new DefaultServer({
    networks,
    defaultHandler: (req, res, next) => {
        res.errors.nxDomain();
    }
});