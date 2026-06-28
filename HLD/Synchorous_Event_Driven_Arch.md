# **Synchronous & Event Driven Architecture**

1. Synchronous Architecture

In synchronous communication, one service waits for another service to finish before continuing.

Example:

Client
   │
   ▼
Order Service
   │
   ▼
Payment Service
   │
   ▼
Success
   │
   ▼
Order Service
   │
   ▼
Client

The client waits until the payment is complete.

Example
Buy iPhone

↓

Order Service

↓

Payment Service

↓

Payment Successful

↓

Order Created

↓

Response to User

The user gets an immediate response.

Disadvantages

Suppose Payment Service is slow.

Order Service

↓

Waiting...

Waiting...

Waiting...

Problems:

❌ Higher latency
❌ Services become tightly coupled
❌ One slow service slows everything
❌ If Payment Service is down, Order Service may also fail

All component of synchronous architecture must scale together.
Consumer needs to resend TX for reporocessing.

2. Event-Driven (Asynchronous) Architecture

Instead of directly calling another service, a service publishes an event.

Order Service

↓

Order Created Event

↓

Message Broker

↓

Payment Service
Inventory Service
Email Service
Analytics Service

Order Service does not wait.

Example

Customer places an order.

Client

↓

Order Service

↓

Save Order

↓

Publish Event

↓

Return Response

The user immediately receives:

202 Accepted

(or another suitable response, depending on the API design)

Then, in the background:

Payment Service

↓

Inventory Service

↓

Notification Service

All work independently.
Message Broker

Usually a broker sits between services.

Examples:

Apache Kafka
RabbitMQ
Amazon SQS
Azure Service Bus

Architecture:

Order Service
      │
      ▼
Message Broker
   ┌──┼───────────┐
   ▼  ▼           ▼
Payment Inventory Email

The broker delivers events to interested consumers.

Real Example

Amazon order:

Without Event-Driven:

Order

↓

Payment

↓

Inventory

↓

Email

↓

Loyalty Points

↓

Analytics

↓

Return Response

The customer waits for every step.

With Event-Driven:

Order

↓

Publish OrderPlaced Event

↓

Response to Customer

Meanwhile:

Payment Service

Inventory Service

Email Service

Analytics Service

They all process the event independently.

Comparison
Synchronous	                        Event-Driven
Caller waits	                    Caller doesn't wait
Immediate response after processing	Immediate acknowledgement, processing continues
Tight coupling	                    Loose coupling
Higher latency	                    Lower perceived latency
Easier debugging	                More complex debugging
Cascading failures possible	        Better fault isolation
Direct API calls	                Events via broker

When to use Synchronous?

Use when the client needs an immediate answer.

Examples:

User Login
Check Balance
Product Search
Payment Authorization
OTP Verification
When to use Event-Driven?

Use when work can happen in the background.

Examples:

Send Email
Send SMS
Generate Invoice
Analytics
Inventory Updates
Recommendation Engine
Notifications

Real Hybrid Architecture

Most large systems use both.

Client
    │
    ▼
Order Service
    │
    ├────────► Payment API (Synchronous)
    │
    ▼
Publish OrderPlaced Event
    │
    ▼
Kafka / RabbitMQ
    │
 ┌──┼─────────────┐
 ▼  ▼             ▼
Email Inventory Analytics

Payment is synchronous because you need to know whether it succeeded before confirming the order.

Email and analytics are asynchronous because they don't have to finish before responding to the user.


Pros:
Each component can scale independently.
Retry built in.

Ex: In Order systerm, Order insert can be done event-driven while order status retrieval can be synchronous.



## Queue and PubSub

First understand the Message Broker

A broker sits between services.

Producer
    │
    ▼
Message Broker
    │
    ▼
Consumers

Examples:

RabbitMQ
Apache Kafka
Amazon SQS
Azure Service Bus

Now there are two communication patterns.

### 1. Queue (Point-to-Point)

A message is processed by only ONE consumer.

          Queue
      ┌──────────┐
Producer │ Order  │
         └──────────┘
              │
      ┌───────┴────────┐
      ▼                ▼
 Worker1          Worker2

Suppose 5 messages arrive.

Order1
Order2
Order3
Order4
Order5

Processing:

Worker1

Order1
Order3
Order5

---------------

Worker2

Order2
Order4

Notice:

Every message is consumed exactly once.

No duplication.

Use Cases
Image Processing
PDF Generation
Video Encoding
Payment Processing
Background Jobs

Example

User uploads an image.

User

↓

Upload Service

↓

Queue

↓

Image Processor

If 1000 images arrive,

just add more workers.

Queue Characteristics
One Producer
One Queue
Multiple Workers
One message → One worker

### 2. Pub/Sub (Publish Subscribe)

A message is delivered to multiple subscribers.

               Topic

         OrderPlaced

              │
      ┌───────┼─────────┐
      ▼       ▼         ▼
 Payment   Email   Analytics

One event:

Order Placed

is received by:

Payment
Inventory
Email
Analytics

Everyone gets a copy.

Example

Customer places an order.

Order Service

↓

Publish OrderPlaced Event

↓

Kafka Topic

Subscribers:

Email Service

↓

Send Email

----------------

Inventory Service

↓

Reduce Stock

----------------

Analytics

↓

Update Dashboard

Same event.

Different consumers.

Queue vs Pub/Sub
Queue	                    Pub/Sub
One consumer	            Multiple consumers
Message processed once	    Every subscriber gets a copy
Point-to-Point	            One-to-Many
Work distribution	        Event broadcasting
Good for background jobs	Good for microservices

Visual Difference
Queue
Producer

↓

Queue

↓

Worker1

or

Worker2

or

Worker3

Only one worker gets each message.

Pub/Sub
Producer

↓

Topic

↓

Consumer A

Consumer B

Consumer C

Everyone gets the event.

Real World Example
Queue

Uber ride request.

Ride Request

↓

Queue

↓

One Driver

Only one driver should accept.

Pub/Sub

Amazon Order.

Order Placed

↓

Topic

↓

Email

Inventory

Payment

Analytics

Recommendation

Everyone needs the event.

Which technologies support what?
Technology	            Queue	                Pub/Sub
RabbitMQ	            ✅	                    ✅
Kafka	                ❌ (Traditional Queue)	✅ Excellent
Amazon SQS	            ✅	                    ❌
Amazon SNS	            ❌	                    ✅
Azure Service Bus Queue	✅	                    ❌
Azure Service Bus Topic	❌	                    ✅


Interview Question ⭐

When should I use Queue?

Answer:

When a task should be processed only once by a single worker.

Examples:

Payment Processing
Email Sending Job
Video Processing
File Upload

When should I use Pub/Sub?

Answer:

When multiple independent services need to react to the same event.

Examples:

Order Placed
User Registered
Payment Completed
Product Added

Interview Diagram ⭐⭐⭐⭐⭐
QUEUE

Producer
    │
    ▼
 Queue
    │
 ┌──┴─────┐
 ▼        ▼
W1       W2

One message
↓

One worker
PUB/SUB

Producer
    │
    ▼
 Topic
 ┌──┼──────────┐
 ▼  ▼          ▼
A   B          C

One event

↓

Everyone receives it

Where do they fit in HLD?

A common architecture uses both:

Client
   │
   ▼
Order Service
   │
   ├── Synchronous call → Payment Service
   │
   ├── Queue → Generate Invoice (one worker)
   │
   └── Pub/Sub → OrderPlaced Event
                 │
        ┌────────┼────────┐
        ▼        ▼        ▼
     Email   Inventory  Analytics


### "Does a message get deleted from a queue once processed?"

A good answer is:

Yes, but only after successful processing. Typically, the consumer processes the message and then sends an acknowledgement (ACK). Once the broker receives the ACK, it deletes the message. If the consumer fails before acknowledging, the message remains (or becomes visible again after a visibility timeout) so another consumer can retry processing. This design helps prevent message loss.

After a timeout (called the visibility timeout in systems like Amazon SQS), the message becomes visible again.

Queue

Order #101

↓

Another Worker

This ensures the message is not lost.

### When message get deleted in pub sub?

The message cannot be considered fully processed until each subscriber has had a chance to receive and acknowledge it according to the broker's delivery model.

What happens after ACK?

Each subscriber maintains its own progress.

Example:

OrderPlaced

↓

Email ACK ✅

↓

Inventory ACK ✅

↓

Analytics ❌

The broker will retry delivery only for Analytics.

Email and Inventory will not receive it again (assuming successful acknowledgment).

Kafka Example

In Apache Kafka:

Topic

OrderPlaced

Consumer Groups:

Email Group

Offset = 100

-----------------

Inventory Group

Offset = 100

-----------------

Analytics Group

Offset = 99

Email and Inventory have processed the message.

Analytics hasn't yet.

Kafka doesn't immediately delete the message after it's consumed.

Instead, it retains messages for a configured retention period (for example, several days), and each consumer group tracks its own offset (the position of the last processed message).

This is one of Kafka's biggest differences from a traditional queue.


RabbitMQ Pub/Sub

In RabbitMQ's publish/subscribe model:

Producer

↓

Exchange

↓

Queue A

Queue B

Queue C

Each subscriber typically has its own queue.

When a subscriber ACKs:

Queue A

↓

Delete Message

Only Queue A's copy is removed.

Queues B and C still keep their copies until their respective consumers acknowledge them.

### Queue vs Pub/Sub Deletion
Queue	                                    Pub/Sub
One copy of the message	                    Each subscriber has its own delivery state (or queue/offset)
One ACK deletes the message	                Each subscriber ACKs independently
One consumer processes it	                Multiple consumers process it
Message removed after successful processing	Removed/marked processed independently for each subscriber (implementation depends                                      on the broker)

"When is a message deleted in Pub/Sub?"

A strong answer is:

"Unlike a queue, where one consumer processes and acknowledges a single message, in Pub/Sub each subscriber processes the event independently. Each subscriber acknowledges its own delivery. The broker tracks acknowledgements separately for each subscriber. The exact deletion behavior depends on the messaging system—for example, RabbitMQ removes the message from each subscriber's queue after it is acknowledged, whereas Kafka retains messages for a configurable retention period and consumers simply advance their offsets."

