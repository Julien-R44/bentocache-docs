# Cache drivers

Some options are common to all drivers. For more information about them, see the [options](./options.md) page. Here we will rather list the specifics of each driver.

## Redis

The Redis driver can be used with many different providers:
- Upstash
- Vercel KV
- DragonFly
- Redis Cluster

The driver uses the [ioredis](https://github.com/redis/ioredis) library under the hood. So all possible ioredis configurations are assignable when creating the bentocache driver. Feel free to look at their documentation for more details.

```ts
import { BentoCache } from 'bentocache'
import { redisDriver } from 'bentocache/drivers/redis'

const bento = new BentoCache({
  default: 'redis',
  stores: {
    redis: redisDriver({ 
      connection: { host: '127.0.0.1', port: 6379 } 
    })
  }
})
```

It is also possible to directly pass an Ioredis instance to reuse the connection.

```ts
import { Redis } from 'ioredis'

const ioredis = new Redis()

const bento = new BentoCache({
  default: 'redis',
  stores: {
    redis: redisDriver({ connection: ioredis })
  }
})
```

You will also need to install `ioredis` to use this driver.

## Filesystem

The filesystem driver will store your cache in a distributed way in several files/folders on your filesystem.

```ts
import { BentoCache } from 'bentocache'
import { fileDriver } from 'bentocache/drivers/filesystem'

const bento = new BentoCache({
  default: 'file',
  stores: {
    file: fileDriver({ directory: 'cache' })
  }
})
```

## Memory

The memory driver will store your cache directly in memory with an [LRU cache algorithm](https://github.com/sindresorhus/quick-lru).

```ts
import { BentoCache } from 'bentocache'
import { memoryDriver } from 'bentocache/drivers/memory'

const bento = new BentoCache({
  default: 'memory',
  stores: {
    memory: memoryDriver({ 
      maxSize: 1000,
    })
  }
})
```

`maxSize` is the maximum number of entries that the cache can contain. If the cache is full, the least used entries will be deleted to make room for new entries.

## DynamoDB

DynamoDB is also supported by bentocache. You will need to install `@aws-sdk/client-dynamodb` to use this driver.

```ts
import { BentoCache } from 'bentocache'
import { dynamoDbDriver } from 'bentocache/drivers/dynamodb'

const bento = new BentoCache({
  default: 'memory',
  stores: {
    memory: dynamoDbDriver({ 
      endpoint: '...',
      region: 'eu-west-3',
      table: {
        name: 'cache' // Name of the table
      },
      credentials: {
        accessKeyId: '...',
        secretAccessKey: '...',
      },
    })
  }
})
```

You will also need to create a DynamoDB table with a string partition key named `key`. You must create this table before starting to use the driver.

:::warning
Be careful with the `.clear()` function of the DynamoDB driver. We do not recommend using it. Dynamo does not offer a "native" `clear`, so we are forced to make several API calls to: retrieve the keys and delete them, 25 by 25 (max per `BatchWriteItemCommand`).

So using this function can be costly, both in terms of execution time and API request cost. And also pose rate-limiting problems. Use at your own risk.
:::

## Cloudflare KV

Keep or not?

## Databases

We offer several drivers to use a database as a cache. Under the hood, we use [Knex](https://knexjs.org/). So all Knex options are available, feel free to check out the documentation.

All SQL drivers accept the following options:

- `connection`: Knex connection options. Or you can directly pass your own Knex instance to reuse an existing connection.
- `autoCreateTable`: If the cache table should be automatically created if it does not exist. Defaults to `true`.
- `tableName`: the name of the table that will be used to store the cache. Default is `bentocache`.

### PostgreSQL

You will need to install `pg` to use this driver.

```ts
import { BentoCache } from 'bentocache'
import { postgresDriver } from 'bentocache/drivers/postgres'

const bento = new BentoCache({
  default: 'mysql',
  stores: {
    mysql: postgresDriver({ 
      connection: { 
        user: 'root', 
        password: 'root', 
        database: 'postgres', 
        port: 5432 
      },
    })
  }
})
```

### MySQL

You will need to install `mysql2` to use this driver.

```ts
import { BentoCache } from 'bentocache'
import { mysqlDriver } from 'bentocache/drivers/mysql'

const bento = new BentoCache({
  default: 'mysql',
  stores: {
    mysql: mysqlDriver({ 
      connection: { 
        user: 'root', 
        password: 'root', 
        database: 'mysql', 
        port: 3306
      },
    })
  }
})
```

### SQLite

You will need to install `better-sqlite3` to use this driver.

```ts
import { BentoCache } from 'bentocache'
import { sqliteDriver } from 'bentocache/drivers/sqlite'

const bento = new BentoCache({
  default: 'mysql',
  stores: {
    mysql: sqliteDriver({ 
      connection: { filename: 'cache.sqlite3' },
    })
  }
})
```
