# adonis-rabbit

`rabbitmq-adonis-v6` is a RabbitMQ provider for [Adonis](https://adonisjs.com/).

## Getting Started

Install `edd-rabbitmq-adonis-v6`:

```
npm i edd-rabbitmq-adonis-v6
```

Then:

```
node ace configure rabbitmq-adonis-v6
```

This is will create `start/rabbit.ts`.

This will create `config/rabbit.ts` and add the following fields to your `.env`:

```
RABBITMQ_HOSTNAME=
RABBITMQ_USER=
RABBOTMQ_PASSWORD=
RABBITMQ_PORT=
RABBITMQ_PROTOCOL= "amqp" //or ampqs
```

Make sure to set the correct values to the enviroment variables so `rabbitmq-adonis-v6` can connect.

## Basic Usage

### Sending messages to an queue

```ts
import { Rabbit } from 'rabbitmq-adonis-v6'
import router from '@adonisjs/core/services/router'

Route.get('/', async () => {
  // Ensures the queue exists
  await Rabbit.assertQueue('my_queue')

  // Sends a message to the queue
  await Rabbit.sendToQueue('my_queue', 'This message was sent by adonis-rabbit')
})
```

### Subscribing

Notice doesn't really makes sense to subscribe to an queue inside a controller, usually this is done through a preload file.

### Listening to an queue

Inside `start/rabbit.ts`:

```ts
import { Rabbit } from 'rabbitmq-adonis-v6'

async function listen() {
  await Rabbit.assertQueue('my_queue')

  await Rabbit.consumeFrom('my_queue', (message) => {
    console.log(message.content)
  })
}

listen()
```

This will log every message sent to my queue `my_queue`.

## Documentation

### RabbitMQ Manager

#### Import

```ts
import { Rabbit } from 'rabbitmq-adonis-v6'
```

#### `assertQueue()`

```ts
await Rabbit.assertQueue('myQueue')
```

Assert the queue is created.

Parameters:

1. `queueName`: the name of the queue
2. `options?`: the queue options

#### `assertExchange()`

```ts
await Rabbit.assertExchange('myQueue', 'type')
```

Assert the exchange is created.

Parameters:

1. `queueName`: the name of the queue
2. `type`: the type of the exchange
3. `options?`: the queue options

#### `bindQueue()`

```ts
await Rabbit.bindQueue('myQueue', 'myExchange', '')
```

Binds a queue and an exchange
.

1. `queueName`: the name of the queue
2. `exchangeName`: the name of the exchange
3. `pattern?`: the pattern (default to `''`)

#### `sendToQueue()`

```ts
await Rabbit.sendToQueue('myQueue', 'content')
```

Parameters:

1. `queueName`: the name of the queue
2. `content`: the content to be send to the queue
3. `options`: the options

Notice that the `content` parameter don't need to be a Buffer, Adonis RabbitMQ will automatically convert it to a Buffer if it isn't already.

You also don't have to `JSON.stringify` an object, Adonis RabbitMQ will also do that for you (it'll be transformed to JSON then to Buffer).

#### `sendToExchange()`

```ts
await Rabbit.sendToExchange('myExchange', 'myRoutingKey', 'content')
```

Parameters:

1. `exchangeName`: the name of the exchange
2. `routingKey`: the routing key
3. `content`: the content to send to the exchange
4. `options`: the options

Notice that the `content` parameter doesn't need to be a Buffer, Adonis RabbitMQ will automatically convert it to a Buffer if it is'nt already.

You also don't have to `JSON.stringify` an object, Adonis RabbitMQ will also do that for you (it'll be transformed to JSON then to Buffer).

#### `consumeFrom()`

```ts
await Rabbit.consumeFrom('myQueue', (message) => {
  console.log(message.content)
  message.ack()
})
```

Consumes a message from a queue.

1. `queueName`: the name of the queue
2. `onMessage` the callback which will be executed on the message receive.

The `onMessage` callback receives a <a href="#message">`Message`</a> instance as parameter.

#### `await ackAll()`

```ts
await Rabbit.ackAll()
```

Acknowledges all the messages.

#### `await nackAll()`

```ts
await Rabbit.nackAll()
```

Rejects all the messages.

Parameters:

1. `requeue?` adds the rejected messages to queue again.

#### `getConnection()`

Retrieves the amqplib's Connection instance. If there`s not a connection, it'll be created.

```ts
await Rabbit.getConnection()
```

#### `getConnection()`

Retrieves the amqplib's Connection instance. If there`s not a connection, it'll be created.

```ts
await Rabbit.getConnection()
```

#### `getChannel()`

Retrieves the amqplib's Channel instance. If there's not a connection, it'll be created. If there`s not a channel, it'll be created too.

```ts
await Rabbit.getChannel()
```

#### `validateConnection()`

Indicate if has an active connection

```ts
await Rabbit.validateConnection()
```

#### `closeChannel()`

Closes the channel.

#### `closeConnection()`

Closes the connection.

---

### Message

When consuming messages through [`consumeFrom`](https://github.com/mvargaslandolfi/rabbitmq-adonis-v6#consumefrom), you'll receive in the callback a Message instance.

This slightly different from amqplib approach. For example:

```ts
Rabbit.consumeFrom('queue', (message) => {
  // Acknowledges the message
  message.ack()

  // Rejects the message
  message.reject()

  // The message content
  console.log(message.content)

  // If you're expecting a JSON, this will return the parsed message
  console.log(message.jsonContent)
})
```

#### `content`

```ts
message.content
```

Returns the message content.

#### `jsonContent`

```ts
message.jsonContent
```

If the message is expected to be in JSON format, then you can use `message.jsonContent` to get the message parsed as an object.

#### `fields`

```ts
message.fields
```

The message fields.

#### `properties`

```ts
message.properties
```

The message properties.

#### `ack()`

```ts
message.ack()
```

Acknowledges the message.

1. `allUpTo?` acknowledges all the messages up to this.

#### `nack()`

```ts
message.nack()
```

Rejects the message.

Parameters:

1. `allUpTo?` rejects all the messages up to this.
1. `requeue?` adds the rejected messages to Queue again.

#### `reject()`

```ts
message.nack()
```

Rejects the message, equivalent to `nack`, but works in older versions of RabbitMQ where `nack` does not.

Parameters:

1. `requeue?` adds the rejected messages to Queue again.

## Usage with Docker

In your .env you have to put the service container name, for example:

`RABBITMQ_HOSTNAME={your_docker_container_service_name}`

## Credits

Part of the code for this package has been taken and adapted from the RabbitMQ package for Adonis v5. You can find the original package [here](https://github.com/jotaajunior/adonis-rabbit).
