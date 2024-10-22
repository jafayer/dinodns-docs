---
sidebar_position: 1
title: Requests & Responses
---

Requests and responses are the classes you'll most interact with in your handlers. Like Express, DinoDNS gives you access to the full request and response object for each handler in the chain. Modifications are allowed to both, allowing you to rewrite parts of the request or construct the response.

:::warning

Like Express, attempting to send more than one response to the client or attempting to modify the response after it has been sent will throw an error. Guard against this by checking `res.finished` before sending a response.

:::

## Interface

The request and response objects are made available to your route handler when you register it:

```ts title="handler.ts"
server.handle('example.com', (req, res) => {
    const {name, type} = req.packet.questions[0];
    // do something with the response
});
```

The request and response objects each have a `packet` property that exposes access to the `dns-packet`'s `Packet` structure. This allows you to read data (such as the questions from the request packet) and write data (such as answers to the response).

:::info
It helps to be familiar with the [dns-packet](https://www.npmjs.com/package/dns-packet) library, as many handlers will involve reading some data from the packets directly.
:::

Request and response also expose access to a `connection` property which exposes some information about the client and the type of connection.

:::tip
The response object is also an event emitter that emits a `done` event when it answers a query via one of the methods discussed below. You can subscribe to this event to dispatch an action asynchronously in a handler after a response has been sent back to a client. This is especially useful for logging purposes.

For more information about how this is accomplished, see the [ConsoleLogger class](https://github.com/jafayer/DinoDNS/blob/main/src/plugins/loggers/ConsoleLogger/index.ts).
:::

For more information, visit the [API reference](https://api.dinodns.dev).

## Answering Queries

There are two ways to respond to a request: `res.answer` and `res.resolve`.

Answers provided in either way are statically type-checked at compile time, ensuring your responses are well-formed and their data types match the types expected by the dns-packet library.

:::danger
There is **no runtime type checking** currently built into the library, so if you have zone data read from I/O it is still possible that runtime type errors will occur.
:::

### `res.answer`

Most of the time, `res.answer` will be the cleaner way to answer your queries. `res.answer` accepts one or more [dns-packet](https://www.npmjs.com/package/dns-packet) `Answer` objects, and automatically sends the response back to the server and marks the response as finished. Your following handlers will still be executed, they just won't be able to respond to the request.

To answer with a single record, pass it in as an object:

```ts
res.answer({
    name: 'example.com',
    type: 'A',
    data: '127.0.0.1'
});
```

To answer with more than one answer, pass in an array of answers:

```ts title="handler.ts"
res.answer([
    {
        name: 'example.com',
        type: 'A',
        data: '127.0.0.1'
    },
    {
        name: 'example.com',
        type: 'A',
        data: '127.0.0.2'
    }
]);
```

### `res.resolve`

`res.resolve` takes no arguments and simply dispatches the response back to the client as-is. This is useful if you've manually set the underlying `res.packet` (such as overwriting the `res.packet.answers` directly).

```ts title="handler.ts"
res.packet.answers = [
    {
        name: 'example.com',
        type: 'A',
        data: '127.0.0.1'
    },
    {
        name: 'example.com',
        type: 'A',
        data: '127.0.0.2'
    }
]

res.resolve()
```
:::warning
For safety (and to support runtime errors when attempting to modify an already-sent response), `res.packet.answers` is a **readonly array**. This means that you can override it entirely, but you cannot append to it, remove from it, or modify its children. You can, of course, modify it immutably like such:
```ts
res.packet.answers = [...res.packet.answers, {name, type, data}]
```
:::


## Error responses

If you instead want to send an error response, the `Response` class also has a helpful "errors" object. Calling a function on the `errors` object automatically sets the correct flags in the headers, and sends the response back to the client.

The four supported errors this way are:

```ts title="errors.ts"
res.errors.notImplemented();
res.errors.nxDomain();
res.errors.refused();
res.errors.serverFailure();
```

Like other responses, be careful not to attempt to send a response after calling an error method, as this marks the packet as finished.