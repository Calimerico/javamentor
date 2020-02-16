### Apache Kafka for begginers

#### What is `Apache Kafka`?
`Apache Kafka` is a message broker.
What is message broker and do we need it? Why choose `Kafka` and not some other message broker like `ActiveMQ`, `RabitMQ` or some other?
You can find answers to those questions here and here. In this article we will assume that we do need broker and we chose `Apache Kafka`.

#### How `Kafka` works?
There are 3 important components in `Apache Kafka`:
- Brokers
- Producers
- Consumers

#### How Publish/Subscribe works and why we need topics?
As you can assume producers are producing messages to brokers and consumers are consuming messages from brokers.
Let's imagine simple banking application example so we can easier understand some new concepts.
Banking application is divided into 5 services.
`Service 1` is producing messages related to money transfer, `Service 2` is producing messages related to which links bank clients are
visiting, `Service 3` is consuming messages related to money transfer and `Service 4` and `Service 5` are consuming messages related to what bank clients visit. You can assume that since every consumer can choose subset of messages from broker, we should separate
messages on broker somehow. That is the reason why `topic` exist. This means that producers will send messages that are related to money transfer to topic named `moneyTransfer` and messages related to client visits will be sent to topic `clientVisits`. On the other hand, consumers will subscribe themselfs onlt to topics in which they are interested in. Important thing to understand here is that `topic` does not work like a `queue`. In `queue` if one consumer receive message from broker, no else consumer will ever receive that same message. In `Kafka` `n` consumers can subscribe to topic and all of them will receive message.
