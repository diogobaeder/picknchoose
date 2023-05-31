# picknchoose

Context-based messaging system.

## Introduction

One of the most exciting technologies I've used is [NATS](https://nats.io/) - an incredible messaging system where publishers and consumers can send/receive messages by hierarchical subjects.

But that kept me thinking: what if I could build a similar system but, instead of having only a hierarchical subject-based subscription functionality, I also had a context-based one where consumers only received messages according to a certain set of constraints? The reasoning behind this is so that the messaging consumers would only receive the messages that they're actually going to use, instead of having them receiving every message for that subject/partition and discarding them if they aren't relevant for them, and this would reduce I/O and allow consumers to get to relevant messages faster.

Imagine, for example, that you have a scenario where you have 10 messages waiting to be consumed. Each of them is a JSON object payload. But, because of the values contained in these messages, a certain consumer is only interested in messages 2 and 9. In a classic messaging system, or event stream, the consumer would receive all 10 messages and end up discarding 8 of them. This is a huge waste of network traffic, and the consumer would also waste more time having to hop from one message to the next. Let's say the consumer spends 5ms to read a message as a baseline: if it's interested only in messages 2 and 9, they would waste at least 30ms to get from message 2 to message 9. If, instead, we have the messaging system itself doing the discarding for them, according to a certain context (filters/constraints), then they would reach message 9 much faster because we would cut out the I/O happening between these messages. 
