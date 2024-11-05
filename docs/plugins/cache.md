---
title: Cache
sidebar_position: 2
---

Cache plugins implement a similar interface to storage plugins, but there are a few key differences.

While stores are intended to support both exact and wildcard matches, cache plugins are intended to support only exact matches. This helps keep cache plugins fast for lookups, and keeps the logic for implementing a cache simple.

Store plugins _also_ do not support queries to get more than one record type for a given domain. This is, likewise, in an effort to reduce hierarchical data structures down to a single lookup. For example, the `DefaultCache` plugin stores records in a map with keys `{domain}:{recordType}`. This constrains the cache to O(1) lookups for all queries that might be run against the cache.

```ts title="cache.ts"
const cache = new DefaultCache();
cache.set('example.com', 'A', '127.0.0.1');
cache.append('example.com', 'A', '127.0.0.2');
cache.get('example.com', 'A'); // ['127.0.0.1', '127.0.0.2']
cache.delete('example.com', 'A', '127.0.0.2');
cache.get('example.com', 'A'); // ['127.0.0.1']
cache.delete('example.com', 'A');
cache.get('example.com', 'A'); // null
```

For a more detailed look at `Cache` and `DefaultCache` APIs, checkout the respective documentation:
- [Cache](https://api.dinodns.dev/classes/plugins.cache.Cache.html)
- [DefaultCache](https://api.dinodns.dev/classes/plugins.cache.DefaultCache.html)