# **WebSockets**

WebSockets are commonly used for real-time, bidirectional communication between the client and the server. Chat applications are one of the best examples. Initially connection is established & then connection remains open & client-server both can call each other.

## What problem do WebSockets solve?

Imagine a chat application.

Without WebSockets:

You send message

↓

Server receives

↓

Friend replies

↓

Your app asks every 2 sec

"Any new message?"

↓

"No"

↓

2 sec later

"Any new message?"

↓

"No"

↓

2 sec later

"Any new message?"

↓

"Yes"

This is called polling.

Problems:

❌ Many unnecessary requests
❌ Higher latency
❌ Wasted bandwidth

With WebSockets

A single connection stays open.

Client  <===================>  Server
          Persistent Connection

Now:

Client can send messages anytime.
Server can push messages anytime.

No repeated polling.

Chatbot Example

Suppose you're chatting with ChatGPT.

You

↓

"Hello"

↓

WebSocket

↓

Server

↓

AI generates response

↓

Server pushes response

↓

Your screen updates instantly

The server doesn't wait for you to ask again—it pushes the response over the existing connection.

## Connection Lifecycle
Open Connection

↓

Handshake

↓

Persistent Connection

↓

Messages Flow Both Ways

↓

Connection Closed


Unlike normal HTTP:

Request

↓

Response

↓

Connection Closed


## HTTP vs WebSocket
HTTP	                        WebSocket
Request → Response	            Two-way communication
New connection for each request	One long-lived connection
Client initiates requests	    Client and server can send anytime
Good for REST APIs	            Good for real-time apps


## Real-world Use Cases
💬 Chat applications
📈 Live stock prices
⚽ Live sports scores
🎮 Multiplayer games
🚕 Ride tracking
📊 Live dashboards
🔔 Notifications

Interview Question

Q: Why not use REST for chat?

Answer:

Because REST is request-response. The server can't send data to the client unless the client makes another request.

WebSockets maintain a persistent connection, allowing the server to push messages instantly, making them ideal for chat and other real-time applications.


Interview Diagram ⭐
            Client A
                │
          WebSocket
                │
                ▼
          Chat Server
                │
          WebSocket
                │
                ▼
            Client B


## WebSocket in Microservices
Users
    │
    ▼
Load Balancer
    │
    ▼
WebSocket Server
    │
    ▼
Redis / Kafka
    │
    ▼
Chat Service

For multiple WebSocket servers, a message broker like Redis Pub/Sub or Apache Kafka is often used so a message received by one server can be delivered to users connected to another server.

Example:

User A → WebSocket Server 1

User B → WebSocket Server 2

If A sends a message to B:

Server 1

↓

Redis Pub/Sub

↓

Server 2

↓

User B

First: Single WebSocket Server (Easy)

Suppose there is only one server.

Alice
   │
WebSocket
   │
Chat Server
   │
WebSocket
   │
Bob

Alice sends:

Hi Bob

The server already has an open WebSocket connection to Bob, so it immediately forwards the message.

Easy.

Now imagine 1 million users

One server cannot handle everyone.

So we scale horizontally.

               Load Balancer
                    │
      ┌─────────────┴─────────────┐
      ▼                           ▼
 WebSocket Server 1        WebSocket Server 2

Now users are distributed.

Example:

Alice  ─────────► Server 1

Bob    ─────────► Server 2

This is where the problem starts.

Problem

Alice sends:

Hi Bob

Flow:

Alice

↓

Server 1

Server 1 asks:

"Where is Bob?"

Bob is not connected to Server 1.

He is connected to Server 2.

How does Server 1 know?

Solution: Message Broker

Both servers connect to a shared messaging system like:

Redis Pub/Sub
Apache Kafka
RabbitMQ

Architecture:

             Load Balancer
             /           \
            ▼             ▼
     WebSocket 1     WebSocket 2
            │             │
            └──────┬──────┘
                   ▼
            Redis Pub/Sub


Now see the message flow

Alice sends:

Hi Bob

Step 1

Alice

↓

WebSocket Server 1

Step 2

Server 1 publishes:

User Bob

Message

Hi Bob

to Redis.

Server 1

↓

Redis

Step 3

Redis broadcasts it.

Redis

↓

Server 2

Step 4

Server 2 says:

Bob is connected to me!

So:

Server 2

↓

Bob

Bob instantly receives:

Hi Bob


Why Redis?

Because without Redis:

Server 1

???

Server 2

The servers have no built-in way to communicate with each other.

Redis becomes the bridge.

Real Example

Imagine WhatsApp.

Millions of users.

User A
↓

Server 5

----------------

User B
↓

Server 18

Server 5 cannot directly know every user connected to every other server.

Instead:

Server 5

↓

Message Broker

↓

Server 18

↓

User B

Why not connect servers directly?

Suppose you have:

100 WebSocket Servers

Without a broker:

Every server must communicate with every other server.

That's a huge number of connections and becomes very difficult to manage.

With a broker:

Server 1 ─┐
Server 2 ─┤
Server 3 ─┤
...        ├── Redis/Kafka
Server100 ─┘

Each server only talks to the broker.

Much simpler and more scalable.

Interview Answer ⭐

If asked:

"Why do WebSocket servers need Redis or Kafka?"

A strong answer is:

"In a distributed system, users are connected to different WebSocket server instances. If User A is connected to Server 1 and User B is connected to Server 2, Server 1 cannot directly push a message to User B. Instead, Server 1 publishes the message to a shared message broker like Redis Pub/Sub or Kafka. The server that owns User B's WebSocket connection receives the event from the broker and pushes it to User B."