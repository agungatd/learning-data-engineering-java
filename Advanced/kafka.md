# Kafka The Definitive Guide 2nd Edition


<!-- ## Table of Contents -->


## Chapter 1

## Chapter 2

## Chapter 3 - Kafka Producers: Sending Messages to Kafka

This document explains how applications send data to Kafka. Think of Kafka as a system that helps different applications share information, like a super-fast and organized postal service for digital messages.

### What is a Kafka Producer?

A Kafka Producer is the part of your application responsible for sending data, or "messages," to Kafka. Messages are organized into categories called "topics."

Imagine an online store processing credit card transactions. When a payment is made, the store's application uses a Kafka Producer to send the transaction details (the message) to a specific topic in Kafka. Other applications, like a fraud detection system, can then read these messages from the topic.

### How Messages are Sent

Sending a message through a Kafka Producer involves several steps:

1.  **Creating a ProducerRecord:** This is like creating a letter to send. It must include the topic you want to send the message to and the actual data (the "value"). You can also optionally add a "key" (like a recipient's name, which helps group related messages) and other details.
2.  **Serialization:** Before sending, the Producer needs to convert the data in your message into a format that can be sent over a network (like converting your letter into a digital file). You need to tell the Producer how to do this conversion using a "serializer". Kafka provides built-in serializers for common data types like text and numbers. For more complex data, you might use standard formats like Avro.
3.  **Partitioning:** Topics in Kafka are divided into "partitions". This helps Kafka handle a lot of data efficiently. If you don't specify which partition your message should go to, Kafka uses a "partitioner" to choose one. If you include a "key" in your message, the default partitioner will use it to make sure messages with the same key always go to the same partition. This is important for keeping related messages in order.
4.  **Batching:** The Producer doesn't usually send each message individually. Instead, it groups messages for the same topic and partition into "batches" to send them more efficiently.
5.  **Sending to Kafka:** A separate process within the Producer sends these batches to the Kafka "brokers" (the servers that make up the Kafka system).
6.  **Acknowledgment:** The broker sends a response back to the Producer to confirm if the message was successfully written to Kafka. If there's an error, the Producer might automatically try sending the message again.

### Ways to Send Messages

There are different ways to send messages, depending on how important it is to know if each message arrives successfully:

* **Fire-and-forget:** Send the message and don't wait for a response. This is fast, but you won't know if the message was lost. This is generally not recommended for important data in real applications.
* **Synchronous send:** Send the message and wait for a response before sending the next one. This guarantees you know if each message was successful, but it's much slower.
* **Asynchronous send with a callback:** Send the message and continue doing other things. When Kafka responds, a special function you provide (a "callback") will be triggered to handle the response (like logging success or dealing with errors). This is a common approach for balancing performance and reliability.

### Important Settings for Producers

Kafka Producers have many settings that affect how they behave. Here are some key ones explained simply:

* **`bootstrap.servers`**: This tells the Producer which Kafka brokers to connect to initially. You should list at least two so the Producer can still connect if one broker is down.
* **`acks`**: This setting controls how much confirmation the Producer waits for from the brokers before considering a message sent successfully.
    * `acks=0`: The Producer sends the message and doesn't wait for any confirmation. Fastest, but messages can be lost without you knowing.
    * `acks=1`: The Producer waits for confirmation that the leader broker (the main copy of a partition) has received the message. Safer than `acks=0`, but messages can still be lost if the leader fails before the message is copied to other brokers.
    * `acks=all`: The Producer waits for confirmation that all synchronized copies (replicas) of the partition have received the message. This is the safest option for preventing data loss, but it adds more delay.
    * **Note:** While lower `acks` settings can make the Producer faster, they don't necessarily make the message available to applications reading the data any sooner. Kafka waits for messages to be copied to all synchronized replicas before allowing consumers to read them to ensure consistency.
* **`delivery.timeout.ms`**: This sets the maximum amount of time the Producer will wait for a message to be successfully sent, including any retries.
* **`retries`**: How many times the Producer will try to resend a message if it gets a temporary error.
* **`linger.ms`**: How long the Producer will wait to gather more messages into a batch before sending it. Waiting a little can increase efficiency and throughput.
* **`buffer.memory`**: How much memory the Producer uses to hold messages waiting to be sent. If messages are coming in faster than they can be sent, this buffer can fill up.
* **`compression.type`**: Whether to compress the messages before sending them. This can save network bandwidth and storage space. Different compression types offer different trade-offs between speed and compression ratio.
* **`enable.idempotence`**: Setting this to `true` helps prevent duplicate messages from being written to Kafka, even if the Producer has to retry sending.

### Serializers and Schema Registry

As mentioned, you need to tell Kafka how to convert your data into bytes using serializers. For complex data structures, using a standardized serialization format like Avro is recommended.

When using Avro, you define the structure of your data using a "schema". A helpful tool when working with Avro and Kafka is a "Schema Registry". Instead of putting the entire schema in every message, the Producer sends the schema to the Schema Registry and includes a small identifier for that schema in the message. Applications reading the messages can then use the identifier to get the schema from the registry and correctly understand the data. This makes it easier to manage changes to your data structure over time.

### Partitioning Strategies

While the default partitioner (using the message key to pick a partition) works well for many cases, you can also create your own custom partitioner if you have specific needs. For example, if you have one customer who sends a disproportionately large amount of data, you might create a custom partitioner to send all of their messages to a dedicated partition to avoid overloading others.

### Headers and Interceptors

* **Headers:** You can add extra information about a message that isn't part of the main data itself, like a tag indicating the source of the message.
* **Interceptors:** These allow you to hook into the Producer's process before messages are sent or after acknowledgments are received. This can be useful for things like monitoring or adding standard information to messages.

### Quotas and Throttling

Kafka can limit how fast Producers can send data (and consumers can receive it) to prevent one application from using too many resources and impacting others. This is done through "quotas". If a Producer exceeds its quota, Kafka will "throttle" it, essentially slowing down its requests.

This chapter provides a good overview of how Kafka Producers work and the various ways you can configure them to meet your application's needs, balancing factors like performance, reliability, and data organization. The next chapter will discuss how applications "consume" or read data from Kafka.