# kafka-ts

Current proposal:
```
type KafkaMessage<T> = {
	topic: string
	partition: number
	offset: number
	data: T
}

type MessageBuckets<T> = {
    [partitionKey: string]: Array<KafkaMessage<T>>
}


type ReceiverOptions<T> = {
	(normal receiver options)

    /*
     * This function will return either the parsed message input `T`
     * or an error indicating why the message could not be parsed.
     *
     * If this function throws an exception, the input will be discarded.
     * 
     * This function is intended to be deterministic; it may NOT return a `Promise`.
     *
     * It is recommended to use a library like `io-ts` for input validation.
     *
     */
	filterMap: <InputParseError>(input: any) => Either<InputParseError, T>

    /*
     * This function is used to indicate when messages must be processed in order, and when messages can be processed in parallel.
     *
     * The structure returned by this function is effectively a map of `string` to `Array<T>`, 
     * where each message in a single `Array<T>` must be processed in order 
     * and messages under different keys may be processed in parallel.
     *
     * An example might be a situation where you want to process messages for single vehicle in order, but messages for different vehicles in parallel:
     * ```
     * type VehicleMessage = {
     *   vehicle_id: string
     *   some_data: any
     * }
     *
     * kafka.receive({
     *   ...,
     *   bucketMessages: (messages: Array<KafkaMessage<VehicleMessage>>) => 
     *     messages.reduce({} as MessageBuckets<VehicleMessage>, (buckets: MessageBuckets<VehicleMessage>, message: KafkaMessage<VehicleMessage>) => {
     *       buckets[message.data.vehicle_id] = [
     *         ...buckets[message.data.vehicle_id],
     *         message,
     *       ];
     *
     *       return buckets;
     *   })
     * });
     * ```
     *
     * This function is intended to be deterministic; it may NOT return a `Promise`.
     *
     */
	bucketMessages: (messages: Array<KafkaMessage<T>>) => MessageBuckets<T>

	processMessage: <ProcessingError extends { retriable: boolean }>(message: KafkaMessage<T>): Promise<Either<ProcessingError, T>>
}
```

## Kafka FAQ

### How do I remove messages from my queue?

It's easy to confuse Kafka with a message queuing system, such as SQS, RabbitMQ, or AMQP. In a message queuing system, subscribers will each have their own queue. In that type of system, you may have the ability to remove specific messages from a specific subscriber's queue.  
When working with Kafka, you instead interact with a "consumer group". You aren't able to "remove" specific messages from the consumer group, but you are able to adjust the "offset" of the consumer group for a specific topic using the Kafka admin scripts. If you aren't familiar with those, you can find a [good cheat sheet here](https://medium.com/@TimvanBaarsen/apache-kafka-cli-commands-cheat-sheet-a6f06eac01b)



### I'm getting InvalidRecordException when trying to produce to my topic

If you see an error along the lines of `org.apache.kafka.common.InvalidRecordException: One or more records have been rejected` one of a few things could be happening.

First, check to see if the topic you are producing to has a cleanup strategy that contains `compacted`. Compacted topics require a `key` to be specified when producing to them.

