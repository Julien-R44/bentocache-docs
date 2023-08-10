# Quick setup

You can install Bentocache via your favorite package manager.

:::warning
Bentocache is an ESM-only package. You will also need Node.js 18 or higher.
:::


:::codegroup
```sh
// title: npm
npm i @julr/bento-cache
```

```sh
// title: pnpm
pnpm add @julr/bento-cache
```

```sh
// title: yarn
yarn add @julr/bento-cache
```
:::


## Setup

Once installed, you can configure BentoCache in your application as follows:

```ts
import { BentoCache } from '@julr/bentocache'
import { memoryDriver } from '@julr/bentocache/drivers/memory'
import { redisDriver } from '@julr/bentocache/drivers/redis'

const bento = new BentoCache({
  default: 'myCache',
  list: {
    myCache: memoryDriver({
      maxSize: 10_000,
      ttl: '2h'
    }),

    redis: redisDriver({
      prefix: 'my-app',
      ttl: '6h',
      connection: {
        host: '127.0.0.1',
        port: 6379
      }
    })
  }
})
```

- Here we have defined two stores. One Redis store, and one memory-only store.
- Bentocache supports named stores. This means that in a single application you can have multiple cache stores. You must define one by default. This is the one that will be used when you call methods directly from the bento object like `bento.get(...)`.
- To use a store other than the default one, you will need to explicitly access it via `bento.use(cacheName)`.


## Hybrid setup

```ts
import { BentoCache } from '@julr/bento-cache'
import { memoryDriver } from '@julr/bento-cache/drivers/memory'
import { redisDriver } from '@julr/bento-cache/drivers/redis'

const bento = new BentoCache({
  default: 'hybrid',

  list: {
    hybrid: hybridDriver({
      local: memoryDriver({/* ... */}),
      remote: redisDriver({/* ... */}),
      bus: redisBus({
        connection: { host: '127.0.0.1', port: 6379 },
      }),
    }),
  },
})

await bento.set('user:42', { name: 'jul' })
console.log(await bento.get('user:42'))
```

With this setup, your in-memory cache will serve as the first level cache. If an item is stored in the in-memory cache, Bentocache will not fetch it from Redis, allowing for huge speed gains.

In a multi-instance application, your different in-memory caches will be synchronized using the bus you have configured.

More information on the [hybrid driver here](tbd)

<!-- ## Next steps

### Interacting with the cache

Let's create a simple API that will manage some users.

```ts
import { bento } from './bentocache.js'

export default class UsersController {
  async show(req) {
    const userId = req.params.id

    const users = bentocache.namespace('users')
    const user = users.getOrSet(`${userId}`, '5m', () => {
      return User.find(userId)
    })

    return user
  }
}
```

Multiple things to note here : 

- We are using a namespace. Namespaces are a way to group keys together. In this case, we are grouping all the users in a namespace called `users`. This will allow us to easily invalidate all the users at once later.
- We are using the `getOrSet` method. This method will first try to fetch the user from the cache. If it is not found, it will execute the *factory* and store the result in the cache for 5 minutes.
- The Factory here is just retrieving the user from the database.

So first time this endpoint is called, it will fetch the user from the database, then store it in the cache. Next time the endpoint is called, it will retrieve the user from the cache. 

### Invalidating the cache

Now, let's say we have an endpoint to update a user. We will need to invalidate the cache for this user. Otherwise we will be serving stale data.

```ts
import { bento } from './bentocache.js'

export default class UsersController {
  async show(req) {
    const userId = req.params.id

    const users = bentocache.namespace('users')
    const user = users.getOrSet(userId, '5m', () => {
      return User.find(userId)
    })

    return user
  }

  async update(req) {
    const userId = req.params.id
    const user = await User.find(userId)

    // Update the user in the database
    await user.update(req.body)

    // Invalidate the cache
    const users = bentocache.namespace('users')
    await users.delete(userId)
  }
}
```

As simple as that, we just need to call the `delete` method on the namespace, passing the key we want to delete. 
Note that if you are using the hybrid driver, the `delete` call will notify the other instances to delete the key from their local cache as well. -->
