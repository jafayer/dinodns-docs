---
sidebar_position: 1
title: Introduction
---

DinoDNS is an event-based, pure-TypeScript DNS server authoring framework.

Unlike most other DNS servers, it is not a standalone artifact or application &mdash; instead, it is meant to provide a convenient, familiar API for authoring custom DNS servers.

The core API is based heavily on the [Express](https://expressjs.com) API with some inspiration from [CoreDNS](https://coredns.io). The server provides a router that helps resolve incoming queries to a plugin chain that performs arbitrary actions on the request and response, such as answering the request, logging, etc.

The framework offers two primary interfaces:
1. A lower-level `DNSServer` abstraction, useful if you want to author a server from scratch
2. A higher-level `DinoDNS` implementation built using `DNSServer` which provides useful defaults and a sensible plugin interface

Many applications can make use of the higher-level `DinoDNS` server, which accepts key plugins (such as storage, caching, logging, etc.) directly as part of its setup. You can read more here.