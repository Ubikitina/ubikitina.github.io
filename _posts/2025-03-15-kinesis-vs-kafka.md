---
layout: post
title: "Kinesis vs. Kafka: Real-Time Engine on AWS"
permalink: /2025-03-15-real-time-kafka-vs-kinesis.html
date: 2025-03-15
---

If you've been following my journey, you probably know that **real-time data processing** is something that truly fascinates me. There is something incredibly satisfying about designing systems that react to events the moment they happen, rather than waiting for a batch job to run hours later.

I've had the opportunity to work extensively with real-time architectures recently, and I wanted to share some of the concepts, tools, and decisions that have shaped my understanding of this ecosystem.

So, let's dive into the world of streams!


## Starting at the Source: Apache Kafka

You can't talk about real-time without talking about **Apache Kafka**. It is the de-facto standard for event streaming, a distributed event store and stream-processing platform capable of handling trillions of events a day.

In simple terms, Kafka allows you to publish and subscribe to streams of records, store them in a fault-tolerant way, and process them as they occur. It acts as the central nervous system for many modern data architectures.



### Why 'Kafka'? The Irony Behind the Name
Have you ever wondered why it's called "Kafka"?

Jay Kreps, one of the co-founders, named it after the author **Franz Kafka**. The reasoning was surprisingly simple: Kafka is a system optimized for **writing**, and Jay liked Franz Kafka's work.

Kreps himself has joked about the irony of the choice. He admitted that while the term *"Kafkaesque"* is usually reserved for nightmarish, overly complex bureaucracies (which, let's be honest, debugging a distributed system can sometimes feel like! 😅), his intention was purely about the literary connection.

### A Visual Gem for Learners
When I was first learning Kafka, I struggled to visualize how topics, partitions, and consumers actually worked together. Then I found this absolute jewel of a resource:

👉 **[Gently Down the Stream](http://www.gentlydownthe.stream/)**

It is one of the most visual, didactic, and charming guides I have ever seen. If you are just starting out, or even if you want a refresher, do yourself a favor and check it out. It makes the abstract concepts click instantly.


## Bringing it to the Cloud: The AWS Real-Time Ecosystem

Since I spend most of my days working within **AWS**, I often have to translate these open-source concepts into managed services. When it comes to moving and storing real-time data in the cloud, there are two main contenders you need to know:

1.  **Amazon Kinesis Data Streams:** The "serverless" native option. It’s easy to set up and integrates seamlessly with other AWS services.
2.  **Amazon MSK (Managed Streaming for Apache Kafka):** Basically, AWS runs Kafka for you. You get the full open-source compatibility without the headache of managing the underlying hardware (Zookeeper nodes, brokers, etc.).



## The Debate: Kinesis Data Streams vs. Amazon MSK

A common question I face is: *“Should we use Kinesis or MSK?”*

While Kinesis is often easier to start with, there are specific scenarios where **Amazon MSK** is the superior (or necessary) choice. Here is a technical breakdown of the key differences that might sway your decision:

### Specs
#### 1. Message Size
* **Kinesis Data Streams:** Strict **1 MB** message size limit. If you have large payloads, you have to split them or store them elsewhere (like S3) and pass a reference.
* **Amazon MSK:** Default is 1 MB, but it is **configurable**. You can bump this up significantly (e.g., to 10 MB or more) if your use case requires heavier events.

#### 2. Scaling Mechanism
* **Kinesis Data Streams:** Uses **Shards**. You scale by splitting or merging shards. It's a very AWS-native concept.
* **Amazon MSK:** Uses **Kafka Topics with Partitions**. Scaling usually involves adding partitions to a topic. (Note: You can add partitions, but you generally can't remove them easily!).

#### 3. Encryption
* **Kinesis Data Streams:** Offers TLS in-flight encryption and KMS at-rest encryption.
* **Amazon MSK:** Offers PLAINTEXT or TLS in-flight encryption, along with KMS at-rest encryption.

#### 4. Security & Authorization
This is often the deciding factor for enterprise environments:
* **Kinesis Data Streams:** Relies entirely on **IAM policies** for both Authentication (AuthN) and Authorization (AuthZ).
* **Amazon MSK:** Offers much more flexibility, supporting:
    * Mutual TLS (AuthN) + Kafka ACLs (AuthZ)
    * SASL/SCRAM (AuthN) + Kafka ACLs (AuthZ)
    * IAM Access Control (AuthN + AuthZ)

### Beyond the Specs: When to Choose MSK?

If Kinesis offers that seductive "serverless" ease of management, what pushes a team towards MSK? In my experience, the decision often moves beyond simple specs and into the realm of architectural strategy and legacy.

First, consider where you are coming from and where you might go. If you are **migrating existing workloads** that already speak "Kafka" (whether on-premise or in other clouds), moving to MSK is essentially a lift-and-shift operation. Furthermore, MSK offers a safety net against **vendor lock-in**. By sticking to the standard open-source protocol, you ensure that your architecture remains portable across different clouds, rather than being coupled tightly to a specific AWS interface.

Then there is the question of data behavior and ecosystem. While Kinesis is fantastic for ephemeral data, it has rigid limits on retention (365 days). Kafka, on the other hand, allows you to treat streams almost like a database. Through features like **log compaction or infinite retention**, MSK can act as a persistent system of record in ways Kinesis simply can't. Add to that the vast ecosystem of **Kafka Connect**, which links to virtually any third-party system imaginable, and you realize that while Kinesis is the king of AWS-native speed, MSK is the master of broad, flexible compatibility.

In short, my advice is: lean towards **Kinesis** for new, purely AWS-native projects where speed of implementation is key. But choose **MSK** when you need the specific power, long-term retention, or strategic freedom of the open standard.


## Final Thoughts

Real-time data opens up a world of possibilities, from instant fraud detection to live dashboards. However, the choice between Kinesis and MSK is rarely just a technical one, it’s **strategic**.

While specific constraints like message size, retention limits, or security protocols will guide your immediate evaluation, the final verdict often depends on your company’s broader architectural vision. Are you prioritizing a fully managed, AWS-native ecosystem for speed and simplicity? Or does your roadmap demand the interoperability and multi-cloud freedom that only open standards can provide?

There is no single "best" tool, only the right one for your specific journey and your organization's long-term goals.

I hope this overview helps you navigate these streams a little better!

---

I’d love to hear about your experiences or challenges! Feel free to reach out and chat with me anytime via DM on my [GitHub profile](https://github.com/Ubikitina).* 😊🚀