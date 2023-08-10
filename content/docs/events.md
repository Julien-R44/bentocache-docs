# Events

Throughout its execution, Bentocache emits different events that can be listened to and received on your end. This allows you to do a lot of things, but mainly for monitoring purposes. Imagine a small Grafana dashboard backed by Prometheus that allows you to see the real-time status and behavior of your cache. Wouldn't that be nice?

### Listening to Events

You simply need to use the `on` method of the Bentocache instance to listen to events.

```ts
bento.on('cache:hit', ({ key, value, store }) => {
  console.log(`cache:hit: ${key}, value ${value}, store ${store}`)
})
```

There is also the `once` method that allows you to listen to an event only once.

```ts
bento.once('cache:hit', ({ key, value, store }) => {
  console.log(`cache:hit: ${key}, value ${value}, store ${store}`)
})
```

And finally, the `off` method that allows you to remove a listener.

```ts
const listener = ({ key, value, store }) => {
  console.log(`cache:hit: ${key}, value ${value}, store ${store}`)
}

bento.on('cache:hit', listener)
bento.off('cache:hit', listener)
```

### Using Your Own Emitter

If you already have an event emitter in your application, you can pass it to Bentocache so that it uses it instead of its own emitter.

```ts
const emitter = new EventEmitter()
const bento = new BentoCache({
  emitter, // 👈 
  default: 'memory',
  stores: {
    memory: memoryDriver({}),
  },
})
```

You can also use an emitter that is not an EventEmitter, as long as it implements the `Emitter` interface:

```ts
export interface Emitter {
  on: (event: string, callback: (...values: any[]) => void) => void
  once: (event: string, callback: (...values: any[]) => void) => void
  off: (event: string, callback: (...values: any[]) => void) => void
  emit: (event: string, ...values: any[]) => void
}
```

[Emittery](https://github.com/sindresorhus/emittery) is notably a good alternative to Node.js's `EventEmitter` that is compatible with this interface.


### List of Events

Here is the list of events emitted by Bentocache:

| Event | Payload | Description |
| --- | --- | --- |
| `cache:hit` | `{ key, value, store }` | Emitted when the key is found in the cache. |
| `cache:miss` | `{ key, store }` | Emitted when the key is not found in the cache. |
| `cache:written` | `{ key, value, store }` | Emitted when the key is written to the cache. |
| `cache:deleted` | `{ key, store }` | Emitted when the key is removed from the cache. |
| `cache:cleared` | `{ store }` | Emitted when the cache is emptied. |
| `bus:message:published` | `{ message }` | Emitted when the bus publishes a message to other applications. |
| `bus:message:received` | `{ message }` | Emitted when the application receives a message instructing it to update its cache. |



<!-- | `cache:hit` | Emitted when the key is found in the cache. The arguments are `{ key, value, store }` |
| `cache:miss` | Emitted when the key is not found in the cache. The arguments are `{ key, store }` |
| `cache:written` | Emitted when the key is written to the cache. The arguments are `{ key, value, store }` |
| `cache:deleted` | Emitted when the key is removed from the cache. The arguments are `{ key, store }` |
| `cache:cleared` | Emitted when the cache is emptied. The arguments are `{ store }` |
| `bus:message:published` | Emitted when the bus publishes a message to other applications. |
| `bus:message:received` | Emitted when the application receives a message instructing it to update its cache. | -->
