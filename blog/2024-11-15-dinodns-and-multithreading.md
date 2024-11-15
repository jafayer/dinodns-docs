---
slug: dinodns-and-multithreading
title: DinoDNS v0.0.12
authors: [jafayer]
tags: [updates]
preview: 
---
DinoDNS v0.0.12 is a major release that brings us closer to a stable launch (which we'll be versioning as 0.1.x). In this launch, we've introduced the higher-level `DinoDNS` class that builds on the DefaultServer class for low-configuration DNS servers, and multithreading support!
<!-- truncate -->

As we get closer to a stable release, we'll also be putting out some concrete performance tests. Early load tests suggest promising results compared to prior benchmarks for CoreDNS.

## `DinoDNS` class

The `DinoDNS` class extends off the `DefaultServer` class to provide a lower-configuration approach to building a dns server. It helps you connect your core plugins to your server without needing to reimplement the boilerplate involved in setting the server up.

For more information, check out the [docs for the DinoDNS core module](/core-library/dinodns).

## Multithreading support

v0.0.11 introduced support for multithreading the `DefaultServer` class (and by extension the `DinoDNS` class). Multithreading is provided by Node.js's built-in [cluster](https://nodejs.org/api/cluster.html) API, and provides a per-thread instance of your DinoDNS server scaled across your machine. Early tests indicate a substantial increase in performance for workloads that support this sort of horizontal scaling.

To read more about multithreading support, see the [DefaultServer docs](/core-library/default-server/).

## Breaking interface changes

There are several breaking interface changes introduced in v0.0.11 and v0.0.12. Notably, we've standardized the network interfaces to all use the same basic shape, and to accept an object instead of ordered parameters.
