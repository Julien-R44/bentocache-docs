---
summary: "Discover the Hybrid Driver: a multi-tier caching solution that combines in-memory and distributed caches for optimal performance."
---

# Hybrid Driver

The hybrid driver is a special driver that allows you to have a multi-tier cache. It is very useful when you want to boost even more the performance of your caching system. 

To do that, we generally use a in-memory cache as the first level cache, and a distributed cache as the second level cache. In-memory cache is really fast, but it is limited by the amount of memory available on your server. Distributed cache is slower, but can store a lot more data, and is shared between your different instances.

So by using a hybrid cache, you can have the best of both worlds. Here is a simplified diagram of the flow :

![Bentocache hybrid](content/docs/hybrid-flow.png)

## Setup

We need to define 3 things to setup the hybrid driver :

```ts
import { BentoCache } from 'bentocache'
import { memoryDriver } from 'bentocache/drivers/memory'
import { redisDriver } from 'bentocache/drivers/redis'

const bento = new BentoCache({
  default: 'hybrid',

  stores: {
    hybrid: {
      driver: {
        driver: hybridDriver({
          // A L1 cache
          local: memoryDriver({ maxSize: 10_000 }),

          // A L2 cache with Redis
          remote: redisDriver({
            connection: { host: '127.0.0.1', port: 6379 },
          }),

          // A bus to synchronize the L1 caches between 
          // the different instances
          bus: redisBusDriver({
            connection: { host: '127.0.0.1', port: 6379 },
          }),
        }),
      },
    }
  },
})
```

We have defined a hybrid cache with :
- An in-memory cache with a maximum size of 10 000 items (after that, the oldest items will be dropped)
- A distributed cache using Redis
- A Redis bus to synchronize the in-memory caches between the different instances of your application

Then, usage is the same as any other cache driver. You can use every method you would use on a normal cache driver. Synchronization between instances, writing to the different caches, etc. everything is handled internally by Bentocache.

## Bus

The bus play a crucial role in the hybrid driver. It is used to synchronize the different in-memory caches between the different instances of your application.

Let's try to understand why we need it in the first place. We have an applications with 2 instances running in parallel with PM2.

- `N1` is calling `bento.getOrSet('user:1', () => fetchUser(1))`
- `N1` is saving the result in in-memory cache + distributed cache.

- After some times, `N2` is also calling `bento.getOrSet('user:1', () => fetchUser(1))`
- `N2` check his memory cache, but found nothing. So it fetches data from distributed cache, and saves it in memory cache.

- `N1` received an update for the user model. So we need to invalidate the cache for `user:1`.
- `N1` invalidates the cache for `user:1` in his in-memory cache, and in the distributed cache. Key doesn't exist anymore in both caches.

- However, `N2` is still holding the old value in his in-memory cache. he doesn't know that the key has been invalidated. So next time it search for `user:1` key, it will find it in his in-memory cache, and will return the old value.

See the problem ? That's why the bus is needed.

### How the bus works

The bus is, as the name suggests, just a bus and messaging system. With the `redisBusDriver` we are leveraging Redis Pub/Sub system to send messages between instances. 

Every time a key is invalidated or updated, the instance will notify other ones by sending a message saying "Hey, this key has been invalidated, you should delete it from your cache". Note that we are not sending the new value to other instances, for multiple reasons : 

- Maybe the other instance will never need this key. So let's not waste memory space on this instance. It will fetch the value from the distributed cache if needed.
- We also save network bandwidth and not overload the bus with serialized data of the value.

Bus messages are also encoded using a custom binary format instead of plain JSON. This allows us to save a lot of space and bandwidth. Also, `JSON.stringify()` and `JSON.parse()` are notoriously slow.

### Retry queue strategy

The bus also has a retry queue strategy. If an instance fails to publish a message through the bus, it will be added to a retry queue. As soon as we can publish messages again, we will try to process that queue and send the messages. 

This can be configured through the `redisBusDriver` options as follow :

```ts
redisBusDriver({
  retryQueue: {
    enabled: true,
    maxSize: undefined
  }
})
```

`maxSize` is the maximum number of items that can be stored in the retry queue. If the queue is full, the oldest item will be dropped. If `undefined`, the queue will have no limit.
