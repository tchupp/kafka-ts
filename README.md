# kafka-ts

## Kafka FAQ

### How do I remove messages from my queue?

It's easy to confuse Kafka with a message queuing system, such as SQS, RabbitMQ, or AMQP. In a message queuing system, subscribers will each have their own queue. In that type of system, you may have the ability to remove specific messages from a specific subscriber's queue.  
When working with Kafka, you instead interact with a "consumer group". You aren't able to "remove" specific messages from the consumer group, but you are able to adjust the "offset" of the consumer group for a specific topic using the Kafka admin scripts. If you aren't familiar with those, you can find a [good cheat sheet here](https://medium.com/@TimvanBaarsen/apache-kafka-cli-commands-cheat-sheet-a6f06eac01b)



### I'm getting InvalidRecordException when trying to produce to my topic

If you see an error along the lines of `org.apache.kafka.common.InvalidRecordException: One or more records have been rejected` one of a few things could be happening.

First, check to see if the topic you are producing to has a cleanup strategy that contains `compacted`. Compacted topics require a `key` to be specified when producing to them.

