# Grace periods

## What is a grace period?

When caching, you are always specifying a TTL (time to live) for your cache entries. This is the amount of time that the cache entry will be valid for. Once the TTL has expired, the cache entry will be considered stale and will forget forever.

Then when the cache entry is requested but not found, you will have to fetch the data from the source of truth (database, API, etc...), and store it in the cache again.

But what if your source of truth is down? Wouldn't it be nice to be able to serve stale data for a little while, until your source of truth is back up?

This is what grace periods are for. Basically, you can specify a grace period when you set a cache entry. This grace period will be the duration during which an entry will still be considered as servable, even if it is stale, if some things go wrong.

## How to use grace periods

Grace periods can be configured at global, driver and operations levels. See the [options](./options.md) documentation for more details.

Let's imagine you have a simple controller that fetches a user from the database, and caches it for 10 minutes:

```ts
cache.getOrSet('users:1', '10m', () => {
  return User.find(1)
})
```

Now, let's say your cache is empty. Someone request the user with id 1, but, the database is down because overloaded. Without grace periods, this request will just fail and will display an error to the user. 

Now what if we add a grace period ?

```ts
cache.getOrSet('users:1', '10m', () => {
  return User.find(1)
}, {
  gracePeriod: { enabled: true, duration: '6h', delay: '5m' }
})
```

- First time this code is executed, user will be fetched from database, stored in cache for **10 minutes** with a grace period of **6 hours**.
- **11 minutes later**, someone request the same user. The cache entry is logically expired, but the grace period is still valid.
- So, we try to call the factory again to refresh the cache entry. But oops, **the database is down** ( or factory is failling for any other reasons ). 
- Since we are still in the grace period of 6h, we will serve the stale data from the cache.
- We reconsider the stale cache entry as valid for 5m ( the delay ). In other words, subsequent requests for the same user will serve the same stale data for 5 minutes. This prevents overwhelming the database with multiple calls while it's down or overloaded. 

So, grace period is a practical solution that can help you maintain a positive user experience even during unforeseen downtimes or heavy loads on your data source. By understanding how to implement and configure this feature, you can make your application more resilient and user-friendly.
