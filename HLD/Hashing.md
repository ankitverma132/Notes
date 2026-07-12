# **Hashing**

What is Hashing?

Hashing is a technique that converts an input (called a key) into a fixed-size value called a hash.

Example:

User ID: 12345

↓

Hash Function

↓

Hash Value: 875

In distributed systems, we often use the hash value to decide where to store or retrieve data.

Primary function of hashing is to distribute traffic equally to all servers.
Unique Id of each call can be used as key for hashing.

Now suppose 1 server goes down, we need to remap entire functionality like all request going to that server fail even if there are other servers avilable.

To solve this we can use consistent hashing.


Simple Hashing Example

Suppose:

3 Servers

Server A
Server B
Server C

Hash function:

Hash(key) % 3

Examples:

Product101

Hash = 7

7 % 3 = 1

↓

Server B

Another:

User55

Hash = 11

11 % 3 = 2

↓

Server C

This distributes data across servers.


## Why use hashing?

Better scalability
Better performance
Lower load on individual servers


Challenge with Simple Hashing

Suppose you initially have:

Hash(key) % 3

Servers:

A
B
C

Now traffic increases.

You add:

Server D

The formula changes:

Hash(key) % 4

Now look what happens.

Example:

Before:

User123

Hash = 11

11 % 3 = 2

↓

Server C

After adding one server:

11 % 4 = 3

↓

Server D

The same user now maps to a different server.

This happens for a large percentage of keys, meaning most cached data or partitions need to be moved.

Problems

Adding or removing servers causes:

Huge data movement
Cache misses
Increased database load
Downtime or expensive rebalancing

This is the biggest challenge with simple hashing.

Solution: Consistent Hashing ⭐⭐⭐⭐⭐

Instead of using:

Hash % N

We place both servers and keys on a logical hash ring.

          Server A

     User1

Server C          User5

      Server B

Each key is assigned to the next server clockwise.

Request are routed to the server which are immediat next in clock wise direction.
Now if server A is removed only user 1 request need to be remapped.



Add a New Server

Suppose we add:

Server D

Only the keys that fall between:

Server B

↓

Server D

need to move.

Everything else stays where it is.

Instead of moving almost all keys, only a small subset is redistributed.

### There is no any hashing mechanism where these is no re-mapping. It will happen.


Where is hashing used?
Redis Cluster
Distributed databases
Load balancing
Sharding
CDNs
Distributed caches

Goals:
Uniform distribution of request among servers.
In case any server goes down, min number of remapping should happen.

Real Example

Suppose your AKS application uses Redis.

AKS Pods

↓

Redis Cluster

Redis1

Redis2

Redis3

If Redis becomes full:

Redis4

is added.

With simple hashing:

Nearly all keys move.

With consistent hashing:

Only a small portion of keys move.

## Consistent hashing working:
Imagine a circle numbered from 0 to 99.

             0
        90       10

    80             20

    70             30

        60      40
             50

This is called the hash ring.

Step 1: Place servers on the ring

Each server also has a hash value.

For example:

Hash(Server A) = 15
Hash(Server B) = 45
Hash(Server C) = 80

So the ring looks like:

             0
                |
          A(15)
                |
               20

               30

          B(45)

               50

               60

               70

          C(80)

               90

The server names aren't important. What matters is their hash values.


Step 2: Hash the key

Suppose the user requests:

Product123

Hash function gives:

Hash(Product123) = 35

Now locate 35 on the ring.

             0

         A(15)

      Product (35)

         B(45)

         C(80)
Step 3: Move clockwise

Rule:

Assign the key to the first server you encounter while moving clockwise.

From 35:

35

↓

Move clockwise

↓

45

↓

Server B

So:

Product123 → Server B

Another example:

Hash(User500) = 82

82 is after Server C (80).

Moving clockwise:

82

↓

90

↓

0

↓

15

↓

Server A

The ring wraps around.

So:

User500 → Server A



## Virtual Nodes:

The Problem Without Virtual Nodes

Suppose we have 3 servers.

Their hash values are:

Server A = 10
Server B = 30
Server C = 90

On the ring:

          0
           |
        A(10)

      B(30)

















      C(90)

Notice something?

Server A owns keys from 90 → 10
Server B owns keys from 10 → 30
Server C owns keys from 30 → 90

Server C owns a much larger portion of the ring.

So:

❌ Server C gets much more traffic.
❌ Load is uneven.
Solution: Virtual Nodes

Instead of placing each physical server once, place it multiple times on the ring.

Suppose each server has 3 virtual nodes.

A1  A2  A3
B1  B2  B3
C1  C2  C3

Now the ring looks like:

0

A1

B1

C1

A2

B2

C2

A3

B3

C3

Now the ownership is spread much more evenly.

Which server owns a virtual node?

Example:

A1 → Server A
A2 → Server A
A3 → Server A

B1 → Server B
B2 → Server B
B3 → Server B

Even though there are 9 points on the ring, there are still only 3 physical servers.

If a key maps to:

Hash(Product123)

↓

A2

The request goes to Server A.

Why is this better?

Without virtual nodes:

Server A → 20% traffic
Server B → 10% traffic
Server C → 70% traffic

Very unbalanced.

With virtual nodes:

Server A → 34%
Server B → 33%
Server C → 33%

Much more balanced.

Bigger Server (More Capacity)

Now suppose:

Server A = Normal
Server B = Normal
Server C = Powerful (more CPU/RAM)

We can give Server C more virtual nodes.

Example:

Server A → 100 virtual nodes

Server B → 100 virtual nodes

Server C → 300 virtual nodes

Now the ring contains:

A1 A2 ... A100

B1 B2 ... B100

C1 C2 ... C300

Server C appears more often on the ring.

As a result:

More keys map to Server C.
Server C handles more traffic.
Load is distributed according to each server's capacity.

Visual Example

Normal servers:

Ring

A
B
C

Powerful server:

Ring

A
B
C
C
C
C
C

Because C appears multiple times, it owns more sections of the ring.

Interview Answer ⭐

If an interviewer asks:

"Why do we use virtual nodes in consistent hashing?"

A strong answer is:

"Virtual nodes improve load distribution. Instead of placing each physical server at a single position on the hash ring, we place it at multiple positions. This spreads data more evenly across servers. If some servers have higher capacity, we can assign them more virtual nodes so they naturally receive a larger share of the data and traffic."