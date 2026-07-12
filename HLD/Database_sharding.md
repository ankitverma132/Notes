# **Dastabase Sharding**


Database sharding is a technique used to split a very large database into multiple smaller databases (shards) so that the load is distributed across multiple servers.

Instead of one database storing all users, each database stores only a subset of the data.


Why do we need sharding?

Imagine your application has 100 million users.

With a single database:

Every read and write goes to one server.
CPU, RAM, disk, and network become bottlenecks.
Eventually, vertical scaling (adding more CPU/RAM) becomes expensive or reaches its limit.

## Sharding solves this by distributing the data.

Example

Suppose you have 12 million users.

Without sharding:

                App
                 |
          ----------------
          |   Database   |
          | 12M users    |
          ----------------

Every request hits the same database.

With 3 shards
                  App
                   |
        -------------------------
        |          |           |
     Shard 1    Shard 2     Shard 3
      1-4M        4-8M       8-12M

Each database stores only a portion of the users.

## How does the application know which shard to use?

This is the key question.

The application uses a sharding key.

Common sharding keys:

User ID
Customer ID
Tenant ID
Country
Region


Example using User ID
Shard = userId % 3

If:

User 1001

1001 % 3 = 2

Go to Shard 2
User 1002

1002 % 3 = 0

Go to Shard 0
User 1003

1003 % 3 = 1

Go to Shard 1

Every request follows the same rule.

Another example (range-based)
Users 1-1,000,000
        |
     Shard 1

Users 1,000,001-2,000,000
        |
     Shard 2

Users 2,000,001-3,000,000
        |
     Shard 3

Simple, but if most new users have high IDs, the last shard can become much busier than the others.

## Benefits

✅ Much more storage

Instead of:

1 Database = 2 TB

You can have:

Shard 1 = 500 GB
Shard 2 = 500 GB
Shard 3 = 500 GB
Shard 4 = 500 GB

✅ Better performance

Reads and writes are spread across multiple databases instead of one.

✅ Horizontal scaling

Need more capacity?

Add another shard instead of buying a bigger database server.

## Challenges
1. Joins become difficult

Suppose:

Orders -> Shard 1

Customers -> Shard 3

A query joining them across shards is much harder and slower than within a single database.

2. Cross-shard transactions

If a transaction touches multiple shards:

Transfer money

Account A -> Shard 1

Account B -> Shard 2

Keeping the transaction consistent is more complex than in a single database.

3. Re-sharding

Suppose you initially have:

3 shards

Later you need:

6 shards

Now you must move data between databases while the application is running. This is one of the most challenging operational tasks.

4. Uneven data distribution

If one shard contains very active users or much more data than others, it becomes a hot shard.

Example:

Shard 1 → 2 million users
Shard 2 → 2 million users
Shard 3 → 8 million users  ❌

Shard 3 will experience higher load and may become a bottleneck.


## Sharding vs Replication
Sharding	                            Replication
Splits data across servers	            Copies the same data to multiple servers
Improves storage and write scalability	Improves read scalability and availability
Each shard has different data	        Every replica has the same data


Many large systems use both together.

             App
              |
     ------------------
     |       |        |
   Shard1  Shard2  Shard3
     |        |        |
  Replica  Replica  Replica

Each shard stores different data, and each shard can have one or more replicas for high availability and read scaling.


Interview takeaway

If you're asked, "When would you use sharding?", a strong answer is:

"I'd use sharding when a single database can no longer handle the application's data volume or write traffic. By partitioning data using a sharding key such as user ID or tenant ID, the system can scale horizontally across multiple database servers. However, I'd also consider the trade-offs, such as more complex queries, cross-shard transactions, and the operational complexity of re-sharding."

## Read and write with shards:

Let's use your app as an example.

Suppose you have Product Service with 4 database shards.

Shard 1
Shard 2
Shard 3
Shard 4

Every product has a unique Product ID.

Product ID = 12345

The application always uses the same formula.

shard = productId % 4

Now:

12345 % 4 = 1

So Product 12345 is stored in Shard 1.

First write

User creates product 12345.

App
   |
Calculate

12345 % 4 = 1

Write to Shard 1

Data is saved there.

Later read

Next day another request comes.

GET /products/12345

The application again calculates

12345 % 4 = 1

It doesn't search all databases.

It directly goes to

Shard 1

and fetches it.

Update
PUT /products/12345

Again

12345 % 4 = 1

Update Shard 1.

Delete

Same thing.

DELETE /products/12345

12345 % 4 = 1

Delete from Shard 1.

So every operation (Read, Write, Update, Delete) for that product always goes to the same shard because the input (productId) hasn't changed.

Another example

Suppose

4 shards

Formula

userId % 4
User ID	Calculation	Shard
101	101 % 4 = 1	Shard 1
102	102 % 4 = 2	Shard 2
103	103 % 4 = 3	Shard 3
104	104 % 4 = 0	Shard 0

Whenever user 102 logs in:

102 % 4 = 2

Always.

Whether today, tomorrow, or next year.

Where is this logic written?

Usually inside the application, or sometimes in a routing layer (middleware/proxy).

Client
   |
API
   |
User Service
   |
Shard Resolver
   |
---------------
|  |  |  |
S1 S2 S3 S4

The Shard Resolver is a small piece of code that decides where to send the request.

Example:

int shard = userId % totalShards;

switch(shard){
    case 0: db0.save(user);
    case 1: db1.save(user);
    case 2: db2.save(user);
    case 3: db3.save(user);
}
But what if we add a new shard?

This is where simple modulo breaks.

Initially:

4 shards

12345 % 4 = 1

Later you add another shard.

5 shards

12345 % 5 = 0

Now the application thinks Product 12345 belongs to Shard 0, but the data is actually still sitting in Shard 1.

This is why naive modulo-based sharding isn't used by large systems.

Instead, systems like Amazon DynamoDB, Cassandra, and many distributed databases use consistent hashing, which minimizes how much data needs to move when shards are added or removed.

Interview tip

If an interviewer asks:

"How do reads always go to the correct shard?"

A strong answer is:

"Every request contains the sharding key, such as userId or productId. The application applies the same deterministic sharding algorithm used during writes. Since the input and algorithm are the same, reads, updates, and deletes are routed to the exact same shard. In large-scale systems, consistent hashing is often preferred over simple modulo because it avoids remapping almost all data when the number of shards changes."