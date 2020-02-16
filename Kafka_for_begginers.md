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

As you can assume producers are producing messages to brokers and consumers are consuming messages from brokers.
Let's imagine simple banking application example so we can easier understand some new concepts.
Banking application is divided into 4 services.
`Service 1` is producing messages related to money transfer, `Service 2` is producing messages related to which links bank clients are
visiting, `Service 3` is consuming messages related to money transfer and `Service 4` is consuming messages related to what bank clients visit.
