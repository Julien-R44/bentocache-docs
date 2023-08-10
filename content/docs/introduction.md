# Introduction

Bentocache is a robust multi-tier caching solution for Node.js applications

- 🗄️ Multi-tier caching
- 🔄 Synchronization of local cache via Bus
- 🚀 Many drivers (Redis, Upstash, In-memory, Postgres, Sqlite and others)
- 🛡️ Graceful Retain and timeouts. Serve stale data when the store is dead or slow
- 🔄 Early refresh. Refresh cached value before needing to serve it
- 🗂️ Namespaces. Group your keys by categories.
- 🛑 Cache stamped protection.
- 🏷️ Named caches
- 📖 Well documented + JSDoc annotations everywhere.
- 📊 Events. Useful for monitoring and metrics
- 🧩 Easily extendable with your own driver

## Why Bentocache ? 

There are already caching libraries for Node: `keyv`, `cache-manager`, or `unstorage`. However, I think that we could rather consider these libraries as bridges that allow different stores to be used via a unified API, rather than true caching solutions as such.

Not to knock them, on the contrary, they have their use cases and cool. Some are even "marketed" as such and are still very handy. But yeah, they serve a different purpose.

Bentocache, on the other hand, is a **true caching solution for web applications**. We indeed have this notion of unified access to differents drivers, but in addition to that, we have a ton of features that will allow you to do serious and robust caching.

If we start from this principle, then I believe there is simply no serious alternative to Bentocache in the JavaScript ecosystem. Which is regrettable, because in all other languages there are robust caching libraries. This is why Bentocache was created.

## Quick presentation

Bentocache is a caching solution aimed at combining performance and flexibility. If you are looking for a caching system that can transition from basic use to advanced multi-level configuration, you are in the right place. Here's what you need to know :

### One-level

The one-level mode is a standard caching method. Choose from a variety of drivers such as **Redis**, **In-Memory**, **Filesystem**, **DynamoDB**, and more, and you're ready to go. 

In addition to this, you benefit from many features that allow you to efficiently manage your cache, such as **cache stampede protection**, **grace periods**, **timeouts**, **namespaces**, etc.

### Hybrid
For those looking to go further, you can use the Hybrid driver with his two-levels caching system. Here's basically how it works:

- **Local Cache**: First level cache. Data is stored in memory with an LRU algorithm for quick access
- **Distributed Cache**: If the data is not in the in-memory cache, it is searched in the distributed cache (Redis, for example)
- **Synchronization via Bus**: In a multi-instance context, you can synchronize different local in-memory caches of your instances via a Bus like Redis or RabbitMQ. This method maintains cache integrity across multiple instances

Here is a simplified diagram of the flow :

![Bentocache hybrid](content/docs/hybrid-flow.png)

All of this is managed invisibly for you via Bentocache. The only thing to do is to set up a bus in your infrastructure. But if you need multi-level cache, you're probably already using Redis rather than your database as a distributed cache. So you can leverage it to synchronize your local caches

The major benefit of hybrid mode is that it allows for responses between 2,000x and 5,000x faster. While Redis is fast, accessing RAM is REALLY MUCH faster.

It's a quite common pattern in the business world, and to quote an example, it's what Stackoverflow does, for example. I invite you to read [this article](https://nickcraver.com/blog/2019/08/06/stack-overflow-how-we-do-app-caching/#layers-of-cache-at-stack-overflow) on the same subject, which is very interesting.

## Features

Below is a list of the main features of BentoCache. If you want to know more, you can read each associated documentation page.

### Multi layer caching

Multi-layer caching allows you to combine the speed of in-memory caching with the persistence of a distributed cache. This is the best of both worlds.

### Lot of drivers

Many drivers available to suit all situations: Redis, Upstash, Database (MySQL, SQLite, PostgreSQL), DynamoDB, Filesystem, In-memory (LRU Cache), Vercel KV...

See the [drivers documentation](./cache_drivers.md) for list of available drivers. Also very easy to extend the library and [add your own driver](tbd)

<!-- :::warning
Only a Redis driver for the bus is currently available. We probably have drivers for other backends like Zookeeper, Kafka, RabbitMQ... Let us know with an issue if you are interested in this.
::: -->


### Resiliency

- [Grace period](./grace_period.md): Keep your application running smoothly with the ability to temporarily use expired cache entries when your distributed cache store is down, or when a factory is failing.

- [Cache stamped prevention](./stampede_protection.md): Ensuring that only one factory is executed at the same time.

- [Retry queue](./hybrid.md#retry-queue) : When a application fails to publish something to the bus, it is added to a queue and retried later.

### Namespaces

The ability to create logical groups for cache keys together, so you can invalidate everything at once later :

```ts
const users = bento.namespace('users')

users.set('32', { name: 'foo' })
users.set('33', { name: 'bar' })

users.clear() 
```

### Events

Events are emitted by Bentocache throughout its execution, allowing you to collect metrics and monitor your cache.

```ts
bento.on('cache:hit', () => {})
bento.on('cache:miss', () => {})
// ...
```

See the [events documentation](./events.md) for more information.

### Friendly TTLs

All TTLs can be passed in a human-readable string format. We use [lukeed/ms](https://github.com/lukeed/ms) under the hood. (this is optional, and you can pass a `number` in milliseconds if you prefer)

```ts
bento.getOrSet('foo', '2.5h', () => getFromDb(), {
  gracePeriod: { enabled: true, duration: '30d' }
})
```

### Early refresh

When you cached item will expire soon, you can refresh it in advance, in the background. This way, next time the entry is requested, it will already be computed and so returned to the user super quickly.

```ts
bento.getOrSet('foo', () => getFromDb(), {
  earlyExpiration: 0.8
})
```

In this case, when only 20% or less of the TTL remains and the entry is requested : 

- It will returns the cached value to the user.
- Start a background refresh by calling the factory.
- Next time the entry is requested, it will be already computed, and can be returned immediately.

### Logging

You can pass a logger to Bentocache, and it will log everything that happens. This is useful for debugging, or to send logs to a monitoring tool.

```ts
import { pino } from 'pino'

const bento = new BentoCache({
  logger: pino()
})

```

See the [logging documentation](./logging.md) for more information.

## Prior art and inspirations

- https://github.com/ZiggyCreatures/FusionCache
- https://laravel.com/docs/10.x/cache
- https://github.com/TurnerSoftware/CacheTower
- https://github.com/dotnetcore/EasyCaching
- https://symfony.com/doc/current/components/cache.html
