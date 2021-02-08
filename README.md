# kafka-ts

## FAQ 

### InvalidRecordException

If you see an error along the lines of `org.apache.kafka.common.InvalidRecordException: One or more records have been rejected` one of a few things could be happening.

First, check to see if the topic you are producing to has a cleanup strategy that contains `compacted`. Compacted topics require a `key` to be specified when producing to them.

