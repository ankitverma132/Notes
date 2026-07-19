# **Cap Theorem**

What is CAP Theorem?

CAP theorem states that a distributed system can guarantee only two out of these three properties during a network partition.

The three properties are:

C = Consistency
A = Availability
P = Partition Tolerance

When a network partition occurs, you must choose between Consistency and Availability.


1. Consistency (C)

Every user sees the same, latest data.

Example:

Suppose your bank balance is ₹1000.

Region A

₹1000

Region B

₹1000

You withdraw ₹500.

Immediately:

Region A

₹500

Region B

₹500

Everyone sees the latest value.

This is Consistency.

2. Availability (A)

Every request gets some response, even if it's not the latest data.

Example:

Region A:

₹500

Region B hasn't received the update yet:

₹1000

A user connected to Region B still gets a response:

Balance = ₹1000

The data is stale, but the system remains available.

3. Partition Tolerance (P)

A network failure occurs between servers.

Example:

Region A

X  (Network Broken)

Region B

The two regions cannot communicate.

This is called a network partition.

In distributed systems, partitions are a reality—they can happen due to network outages, router failures, or region isolation.

Why can't we have all three?

Suppose:

Region A

₹500

X

Region B

₹1000

A network partition exists.

Now a user asks Region B:

"What's my balance?"

You have two choices.

Option 1: Return ₹1000
✅ Available (you answered)
❌ Not consistent (stale data)

This is AP.


Option 2: Refuse to answer

Region B replies:

"I can't verify the latest balance."

✅ Consistent
❌ Not available

This is CP.

You cannot simultaneously provide the latest data and always respond when the network is partitioned.


## CP Systems

Choose:

✅ Consistency
✅ Partition Tolerance

Sacrifice:

❌ Availability

Example:

Client

↓

Database

↓

"I cannot serve this request right now."

The system waits until replicas synchronize.

Examples:

Apache ZooKeeper
etcd
HBase


## AP Systems

Choose:

✅ Availability
✅ Partition Tolerance

Sacrifice:

❌ Immediate consistency

Example:

User A

↓

Likes a Post

↓

Region A Updated

↓

Region B Updates Later

For a short time, users in different regions may see different like counts.

Examples:

Cassandra
DynamoDB (configurable)
Riak

## What about CA?

Choose:

✅ Consistency
✅ Availability

No partition.

This works only when there is no network partition.

Example:

Application

↓

Single Database

In a true distributed system, you cannot assume partitions will never happen, so CA isn't generally achievable during a partition.


## Visual Summary
Choice	Meaning
CP	    Correct data, but may reject requests
AP	    Always responds, but data may be stale
CA	    Works only when there is no partition

Real-world Examples
Banking

Need correct balances.

Prefer:

CP

Better to reject a request than show an incorrect balance.

Social Media

Like counts or follower counts.

Prefer:

AP

It's acceptable if counts are slightly delayed.

E-commerce

Often a mix.

Product descriptions → AP is often acceptable.
Payment processing → CP is preferred.

Different parts of the same application can make different trade-offs.

Interview Answer ⭐

If asked:

"Explain CAP Theorem."

A strong answer is:

"CAP theorem states that in a distributed system, when a network partition occurs, it's impossible to guarantee both Consistency and Availability at the same time. Partition tolerance is generally required because network failures are inevitable, so systems typically choose either CP (prioritizing correct data) or AP (prioritizing continuous availability), depending on business requirements."