---
title: Logging
sidebar_position: 3
---

Logging plugins are not intended to modify the request or response in any way, but instead should be able to dispatch a log event synchronously or asynchronously. Loggers can log anything, from metrics about query volumes to individual queries themselves. Loggers can subscribe to events from the `DNSResponse` object and can log data reactively. If you're using the `ConsoleLogger`, this step is already taken care of for you if you configure the logger to log responses:

```ts title="loggedServer.ts"
const server = new DefaultServer({
    networks: [...]
});

s.use(logger.handler);
```

When the server receives a new request that is passed to the logger, if `logRequests` is enabled, it will immediately dispatch a log. If `logResponses` is enabled, the logger will subscribe to the `DNSResponse`'s `done` event, dispatching a separate log once the answer is recorded.

:::warning
Custom or third party loggers may be more complicated in nature than the simple `ConsoleLogger`. If that's the case, it's important to make sure that all logging actions are performed asynchronously. Synchronous logs may degrade server performance.
:::

For a more detailed look at `Cache` and `DefaultCache` APIs, checkout the respective documentation:
- [Logger](https://api.dinodns.dev/interfaces/plugins.logging.Logger.html)
- [ConsoleLogger](https://api.dinodns.dev/classes/plugins.logging.ConsoleLogger.html)