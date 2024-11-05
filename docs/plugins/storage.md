---
title: Storage
sidebar_position: 1
---

A storage plugin is a database that stores zone data. The main `Store` interface is backend agnostic &mdash; stores can be backed by durable data written to disk, a database engine, or even ephemeral, in-memory structures. The `DefaultStore`, for example, is a simple in-memory implementation.

Stores are generally expected to allow for direct lookups and wildcard lookups. They should implement the matching behavior described by [RFC 1034](https://datatracker.ietf.org/doc/html/rfc1034#section-4.3.3). Likewise, they should implement interfaces to retrieve all the data for a particular zone, or just one record. Most stores should also implement methods to manipulate zone data; the interface supports methods to get, set, append, and delete zone data.

```ts title="storage.ts"
const store = new DefaultStore();
store.set('example.com', 'A', '127.0.0.1');
store.append('example.com', 'A', '127.0.0.2'); 
store.delete('example.com', 'A', '127.0.0.1');
store.get('example.com', 'A'); // ['127.0.0.2']
store.delete('example.com', 'A');
store.delete('example.com');
store.get('example.com', 'A') // null
```

For a more detailed look at `Store` and `DefaultStore` APIs, checkout the respective documentation:
- [Store](https://api.dinodns.dev/classes/plugins.storage.Store.html)
- [DefaultStore](https://api.dinodns.dev/classes/plugins.storage.DefaultStore.html)

