# Kafka

## Handle Exceptions using errorChannel

Related

> https://stackoverflow.com/questions/46817524/sending-error-message-to-error-channel-using-spring-cloud-stream
> https://cloud.spring.io/spring-cloud-static/spring-cloud-stream-binder-kafka/2.1.2.RELEASE/single/spring-cloud-stream-binder-kafka.html#kafka-error-channels
> https://github.com/spring-cloud/spring-cloud-stream/issues/1338#issuecomment-378493804

The `application.yaml` should have the following properties activated inside the producer bindings

```yaml
spring:
  cloud:
    stream:
      kafka:

        bindings:
          output-binding:
            producer:
              configuration:
                sync: true
                errorChannelEnabled: true

          custom-channel-error:
            producer:
              configuration:
                sync: true # this is not necessary, it's only for the produder to work on sync
                errorChannelEnabled: true
```

Then, use a `@ServiceActivator`:

```java
...

@ServiceActivator(inputChannel = SPRINGCLOUDCONSTANT.ERROR_CHANNEL_DEFAULT)
public void handle(Message<?> message) {
  ...
}

...
```
