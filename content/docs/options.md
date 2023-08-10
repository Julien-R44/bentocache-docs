# Options

Here at the different possible options of BentoCache. Some of them are configurable, either globally, at the driver level or at operation level ( when calling core methods like `getOrSet`, `get`, etc. ).

Order of precedence is as follows: **Operation level > Driver level > Global level.**

### `prefix`

Default: `undefined`

Levels: `global`, `driver`

This prefix will be added in front of all your cache keys. This can be useful, for example, in the case where you share a Redis with another application. Particularly when you need to `.clear()`. Without a prefix, all keys will be deleted, even those not added by your application.

### `ttl`

Default: `30s`

Levels: `global`, `driver`, `operation`

The TTL of the item to cache.

### `suppressRemoteCacheError`

Default: `false`

Levels: `global`, `driver`, `operation`

In hybrid mode, if `false`, then errors thrown by your remote cache will be rethrown, and you will have to handle them yourself. Otherwise, they will just be ignored.

Note that in some cases, like when you use Graceful Retain, errors will not be thrown, even if this option is set to `false`.

### `earlyExpiration`

Default: `undefined`

Levels: `global`, `driver`, `operation`

Percentage of the TTL (between 0 and 1) that corresponds to the moment from which bentocache should launch a background refresh of your key. For example, if your TTL is `10m`, and your `earlyExpiration` is `0.8`, then a background refresh will be launched when the key is requested and there is less than 2 minutes of TTL remaining.

### `gracePeriod`

Default `undefined`

Levels: `global`, `driver`, `operation`

An object to configure the [grace period](grace_period):
```ts
{
  enabled: true,
  duration: '6h',
  delay: '5m'
}
```

### `retryQueue`

Default: 
```ts
{
  enabled: true,
  maxSize: 1000,
}
```

Levels: `global`

An object to configure the retry queue of the bus. If not enabled, an item that fails to be sent to the bus will be dropped.
`maxSize` is the maximum number of items that can be stored in the retry queue. If the queue is full, the oldest item will be dropped.

### `lockTimeout`

Default: `undefined`

Levels: `global`, `operation`

The maximum amount of time (in milliseconds) that the in-memory lock for [stampeded protection](./stampede_protection.md) can be held. If the lock is not released before this timeout, it will be released automatically.

### `logger`

Default: `undefined`.

Levels: `global`, `driver`

Only configurable at the BentoCache level.

See [logger](./logging.md) for more details.

### `emitter`

Default: `new EventEmitter()`.

Only configurable at the BentoCache level.

See [events](./events.md) for more details.


## TTL Formats

As a reminder, all TTLs accept two different formats:
- a `number` in milliseconds.
- a `string`, duration in human-readable format, see [lukeed/ms](https://github.com/lukeed/ms) for more details on the different formats accepted.

