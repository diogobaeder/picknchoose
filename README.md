# picknchoose

Context-based messaging system.

## Introduction

One of the most exciting technologies I've used is [NATS](https://nats.io/) - an incredible messaging system where publishers and consumers can send/receive messages by hierarchical subjects.

But that kept me thinking: what if I could build a similar system but, instead of having only a hierarchical subject-based subscription functionality, I also had a context-based one where consumers only received messages according to a certain set of constraints? The reasoning behind this is so that the messaging consumers would only receive the messages that they're actually going to use, instead of having them receiving every message for that subject/partition and discarding them if they aren't relevant for them, and this would reduce I/O and allow consumers to get to relevant messages faster.

Imagine, for example, that you have a scenario where you have 10 messages waiting to be consumed. Each of them is a JSON object payload. But, because of the values contained in these messages, a certain consumer is only interested in messages 2 and 9. In a classic messaging system, or event stream, the consumer would receive all 10 messages and end up discarding 8 of them. This is a huge waste of network traffic, and the consumer would also waste more time having to hop from one message to the next. Let's say the consumer spends 5ms to read a message as a baseline: if it's interested only in messages 2 and 9, they would waste at least 30ms to get from message 2 to message 9. If, instead, we have the messaging system itself doing the discarding for them, according to a certain context (filters/constraints), then they would reach message 9 much faster because we would cut out the I/O happening between these messages. 

## Ideas

Here are some ideas for this project. They will change a lot, for now, as I'm still figuring out even what I want specifically.

### Filtering

One of the first ideas I had, for filtering messages, was to use a similar syntax to [MongoDB queries](https://www.mongodb.com/docs/manual/tutorial/query-documents/). They are super flexible and I would leverage a "language" that many engineers already know. Whenever a consumer started consuming from the subject/queue/partition, they would send the filters, and the system would only send them messages that match these filters.

### Indexing

Not really "indexing", but the idea is to have a simplified view of the message to allow faster filtering. Let's say, for example, that a message payload includes not only small values, but also big byte arrays representing files - we don't want to reconstruct the whole message as an object in memory only to filter by some small fields.

For example, suppose we have this message:

```json
{
  "name": "John Doe",
  "age": 25,
  "avatar": "0101001001111001010100111001100111"
}
```

we would have to rebuild the whole message in memory to filter by `age`. If, instead, we have a simplified version of it:

```json
{
  "age": 25
}
```

we would severely reduce the de-serialization cost for it, if the consumer is only interested in filtering by age.

### Subjects

The idea is to do something similar to what NATS does, which is to organize messages by subjects. While we could just go with context-based filtering, that mechanism is still a bit more complex and probably more expensive than sending messages by subject (no, I have no data yet to support this affirmation - but I plan to have once I have some implementation of this).

## Decisions

### Language

The very first decision I have for this project is to use [Rust](https://www.rust-lang.org/), because:
* It's a very efficient/fast language
* It's a very secure language
* It will avoid the issues I would have with any garbage collector-based language (like Go, Java etc)
* It's getting significant adoption by engineers, so I have a higher chance of getting collaboration to the project (this is why I didn't choose Zig, for example)
* I just like the language, and liking some technology is important to keep me motivated

### Durability

The system has to be durable, period. If it goes down for some reason, it has to be able to pick up from where it was and continue to operate - like [Kafka](https://kafka.apache.org/) does, for example. Or RabbitMQ, in the case of [durable queues](https://www.rabbitmq.com/queues.html#durability). Or NATS with [JetStream](https://docs.nats.io/nats-concepts/jetstream).

This obviously means the system would end up being more complex and slower than if we had in-memory messages only, but it's a good trade-off since disaster recovery is important to me.

### Name

I don't have a strong reason behind the name, it just came to my mind and I used it. Plus, it sounds funnily similar to "Pikachu" (I'm not a fan of Pokémon, but I know what a Pikachu is). Since this project is basically a hobby (at the moment), I want to have fun with it, so...
