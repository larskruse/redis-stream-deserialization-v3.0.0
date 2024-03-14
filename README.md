# redis-stream-deserialization

## Example based on 
https://redis.io/topics/streams-intro


## how to run it

```shell
docker-compose down -v && docker-compose up --build
```


## OK scenario

Adding entry with `_class` field-value pair. 
```shell
docker-compose exec redis redis-cli XADD mystream \* sensorId 1234 temperature 19.8 _class pl.lrozek.redis.stream.consumer.domain.TemperatureReadingDto
```
When `_class` field-value pair is present, entry is deserialized properly and following text is logged:
```
2024-03-14T20:38:36.868Z  INFO 1 --- [cTaskExecutor-1] .l.r.s.c.r.l.TemperatureReadingsListener : received following message: ObjectBackedRecord{recordId=1710448716777-0, value=TemperatureReadingDto(sensorId=1234, temperature=19.8)}

```


## not OK scenario
Adding entry without `_class` field-value pair

```shell
docker-compose exec redis redis-cli XADD mystream \* sensorId 1234 temperature 19.8
```
When `_class` field-value pair is missing it caues an exception and entry is not consumed.

```
 2024-03-14T20:38:50.889Z ERROR 1 --- [cTaskExecutor-1] ageListenerContainer$LoggingErrorHandler : Unexpected error occurred in scheduled task

 org.springframework.core.convert.ConversionFailedException: Failed to convert from type [org.springframework.data.redis.connection.stream.StreamRecords$ByteMapBackedRecord] to type [pl.lrozek.redis.stream.consumer.domain.TemperatureReadingDto] for value [org.springframework.data.redis.connection.stream.StreamRecords$ByteMapBackedRecord@5cf5f68]
      at org.springframework.data.redis.stream.StreamPollTask.convertRecord(StreamPollTask.java:178) ~[spring-data-redis-3.0.0.jar:3.0.0]
      at org.springframework.data.redis.stream.StreamPollTask.deserializeAndEmitRecords(StreamPollTask.java:156) ~[spring-data-redis-3.0.0.jar:3.0.0]
      at org.springframework.data.redis.stream.StreamPollTask.doLoop(StreamPollTask.java:128) ~[spring-data-redis-3.0.0.jar:3.0.0]
      at org.springframework.data.redis.stream.StreamPollTask.run(StreamPollTask.java:112) ~[spring-data-redis-3.0.0.jar:3.0.0]
      at java.base/java.lang.Thread.run(Thread.java:1583) ~[na:na]
 Caused by: java.lang.IllegalArgumentException: Value must not be null
      at org.springframework.util.Assert.notNull(Assert.java:172) ~[spring-core-6.1.1.jar:6.1.1]
      at org.springframework.data.redis.connection.stream.Record.of(Record.java:99) ~[spring-data-redis-3.0.0.jar:3.0.0]
      at org.springframework.data.redis.connection.stream.MapRecord.toObjectRecord(MapRecord.java:139) ~[spring-data-redis-3.0.0.jar:3.0.0]
      at org.springframework.data.redis.core.StreamObjectMapper.toObjectRecord(StreamObjectMapper.java:137) ~[spring-data-redis-3.0.0.jar:3.0.0]
      at org.springframework.data.redis.core.StreamOperations.map(StreamOperations.java:566) ~[spring-data-redis-3.0.0.jar:3.0.0]
      at org.springframework.data.redis.stream.DefaultStreamMessageListenerContainer.lambda$getDeserializer$2(DefaultStreamMessageListenerContainer.java:213) ~[spring-data-redis-3.0.0.jar:3.0.0]
      at org.springframework.data.redis.stream.StreamPollTask.convertRecord(StreamPollTask.java:176) ~[spring-data-redis-3.0.0.jar:3.0.0]
      ... 4 common frames omitted
```

## Excpected behaviour
`_class` field-value pair should not be mandatory to propely deserialize entry to `ObjectRecord`. `StreamMessageListenerContainerOptions` configured with `targetType` has already information regarding class used for deserialization. Look at
https://github.com/larskruse/redis-stream-deserialization-v3.0.0/blob/main/redis-stream-consumer/src/main/java/pl/lrozek/redis/stream/consumer/redis/configuration/RedisListenerConfiguration.java#L29

Enforcing producer system, which can be written using any language / framework should not be enforced to add `_class`. 



To inspect stream content
- via CLI `docker-compose exec redis redis-cli XREAD COUNT 10 STREAMS mystream 0-0`
![cli-streamContent](https://user-images.githubusercontent.com/741781/151556112-54271556-0f92-4948-9943-b66235439c08.png)
