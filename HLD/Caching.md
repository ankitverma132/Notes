# **Caching**

What is Caching?

A cache stores frequently accessed data in a faster storage layer so future requests don't have to fetch it from the original source (usually a database).

Without cache:

User

↓

Application

↓

Database

↓

Application

↓

User

Every request hits the database.


Problem

Suppose your product page gets:

1 Million Requests

↓

Database

The database becomes overloaded.

Even though everyone is requesting the same product.



Solution: Cache
User

↓

Application

↓

Redis Cache

↓

Database

Now most requests are served from Redis, which is much faster than querying the database.

Example

User requests:

GET /product/101
First Request
User

↓

Application

↓

Redis

(No Data)

↓

Database

↓

Product

↓

Redis (Store)

↓

User

This is called a Cache Miss.

Second Request
User

↓

Application

↓

Redis

(Product Found)

↓

User

The database is not accessed.

This is called a Cache Hit.

## Cache Hit vs Cache Miss
Cache Hit ✅
Cache

↓

Data Found

Fast response.

Cache Miss ❌
Cache

↓

Not Found

↓

Database

Slower response.

### Cache entries got deleted after a specific time. This time is called TTL.
Cachec is not restricted to backend. You can enable caching to different part of system.

## Where is Cache Used?
Product Catalog
User Profile
Trending Videos
News Feed
Weather Data
Cricket Scores
Configuration Data

Basically, data that is read frequently.


## What Should NOT Be Cached?

Examples:

Bank Balance
Live Stock Trading
OTP Verification
Payment Status

These require fresh, accurate data.

## Types of Cache
1. Client Cache

Browser stores:

Images
CSS
JavaScript
Browser

↓

Cache

↓

No Server Call


2. CDN Cache

Static content is stored closer to users.

User

↓

CDN

↓

Origin Server

Examples:

Images
Videos
CSS


3. Application Cache (Redis)
Application

↓

Redis

↓

Database

Most common in HLD.

Cache Eviction

Cache cannot store everything forever.

Policies decide what to remove.

Common policies:

LRU (Least Recently Used)
LFU (Least Frequently Used)
FIFO (First In First Out)
TTL (Time To Live)

Example:

TTL = 5 minutes

After 5 minutes, the cached item expires.

### Redis

The most common cache in HLD.

Why Redis?

In-memory
Extremely fast
Supports TTL
Supports multiple data structures
Easy to scale



Interview Summary
Concept	        Meaning
Cache	        Temporary fast storage
Cache Hit	    Data found in cache
Cache Miss	    Data not found; fetch from DB
Redis	        Most popular in-memory cache
TTL	            Automatic expiration time
Eviction	    Removing old cache entries

## Real Architecture
Users
   │
   ▼
Load Balancer
   │
   ▼
Application
   │
   ▼
Redis Cache
   │
(Cache Miss)
   ▼
Database

## Typical Azure Architecture
Internet
     │
     ▼
Azure Front Door
     │
     ▼
API Management
     │
     ▼
AKS
     │
 ┌───────────────┐
 │ Product Pods  │
 │ Order Pods    │
 │ User Pods     │
 └───────────────┘
     │
     ▼
Azure Cache for Redis
     │
     ▼
Azure SQL Database


One important principle

Don't store cache inside your Pods.

Pods can be restarted, rescheduled, or scaled up and down at any time. A shared external cache like Azure Cache for Redis is the standard production approach because all Pods can use it consistently.

Interview Answer ⭐

If the interviewer asks:

"Where would you place Redis in an AKS-based architecture?"

A strong answer is:

"I would deploy a shared Redis cache (such as Azure Cache for Redis) between the application running on AKS and the database. All Pods would access the same Redis instance. On a cache hit, the application serves the data directly from Redis. On a cache miss, it reads from the database, returns the data to the client, and stores it in Redis for future requests. This reduces database load and improves response times."



## Redis cache and memcached
The short answer is:

Redis = More features, most commonly used today.
Memcached = Simpler, lightweight, pure caching.

Redis supports more complex use cases in compare to memcached such as advanced data structures, sorting, supports complex data type, transaction pub sub etc.

Redis

Redis is an in-memory data store that can be used as a cache and much more.

Supports:

Strings
Hashes
Lists
Sets
Sorted Sets
Streams
Pub/Sub

Example:

User Profile

↓

Redis

↓

{
  name: "Ankit",
  age: 26
}
Features
✅ Extremely fast
✅ TTL support
✅ Persistence (can save data to disk)
✅ Replication
✅ Pub/Sub
✅ Transactions
✅ Distributed caching
✅ Rich data structures
Memcached

Memcached is a simple in-memory key-value cache.

Example:

Key

product:101

↓

Value

Laptop Details

That's essentially it.

Features
✅ Very fast
✅ Simple key-value cache
✅ Lightweight
❌ No persistence
❌ No Pub/Sub
❌ Limited data structures

Comparison
Feature	         Redis	               Memcached
Primary Purpose	Cache + Data Store	Cache
Data Structures	Many	               Key-Value only
Persistence	      ✅ Yes	               ❌ No
TTL	            ✅ Yes               	✅ Yes
Replication	      ✅ Yes	               Limited
Pub/Sub	         ✅ Yes	               ❌ No
Transactions	   ✅ Yes	               ❌ No
Distributed Locks	✅ Yes	               ❌ No
Memory Usage	   Slightly higher	   Lower
Complexity	      More features	      Very simple

Which one is used more today?

Today, Redis is far more common in cloud-native applications because it can solve multiple problems beyond caching.

For example, in an AKS application:

Users
   │
   ▼
AKS Pods
   │
   ▼
Redis
   │
   ▼
Azure SQL

The same Redis instance might also be used for:

Caching
Session storage
Rate limiting
Distributed locks

Interview Answer ⭐

If asked:

"Redis or Memcached—which would you choose?"

A good answer is:

"For most modern applications, I would choose Redis. It provides very fast in-memory caching but also supports persistence, rich data structures, replication, Pub/Sub, and distributed locking. Memcached is a good choice when I only need a simple, lightweight key-value cache, but Redis is generally more versatile and is the more common choice in cloud-native architectures."


For AKS, microservices, and most modern backend systems, Redis is typically the preferred choice and is the one you're most likely to encounter in HLD interviews.


## How cache populated(Caching Strategies):

1. Cache-Aside (Lazy Loading) ⭐⭐⭐⭐⭐

This is the most widely used strategy.

Read Flow
User
   │
   ▼
Application
   │
   ▼
Redis
Cache Hit
Redis

↓

Data Found

↓

Return to User

Done.

Cache Miss
Application

↓

Redis

(No Data)

↓

Database

↓

Store in Redis

↓

Return to User

The application is responsible for populating the cache.

Example
GET /products/101

First request:

Redis ❌

↓

SQL

↓

Redis

↓

User

Second request:

Redis ✅

↓

User
Pros
Easy to implement
Cache only frequently accessed data
Most common strategy
Cons
First request is slower (cache miss)
Cache can become stale if not invalidated


2. Read-Through

Here, the application doesn't know whether the data is cached.

Application

↓

Cache

↓

Database

If data is missing:

Cache

↓

Database

↓

Store Data

↓

Application

The cache layer itself fetches and stores the data.

Pros
Simpler application code
Cache manages loading
Cons
Requires cache support or additional infrastructure
Less commonly used with Redis directly


3. Write-Through

Whenever data is written:

Application

↓

Redis

↓

Database

Both cache and database are updated before the write is considered successful.

Example:

Update Product Price

↓

Redis Updated

↓

Database Updated
Pros
Cache always has fresh data
Good consistency
Cons
Slower writes
May cache data that is never read


4. Write-Back (Write-Behind)

Writes go to the cache first.

Application

↓

Redis

↓

Success

↓

Later...

↓

Database

The database update happens asynchronously.

Pros
Very fast writes
Good for high write throughput
Cons
Risk of data loss if the cache fails before writing to the database
More complex


5. Refresh-Ahead

Before cached data expires:

Redis

TTL Almost Over

↓

Refresh from Database

↓

Reset TTL

Popular data stays warm in the cache.

Pros
Avoids cache misses for frequently accessed data
Consistently low latency

Cons
May refresh data that's no longer needed

Which strategy is used most?

For applications running on AKS with Azure Cache for Redis and Azure SQL Database, the most common pattern is Cache-Aside.

Example:

Users
   │
   ▼
AKS Pods
   │
   ▼
Redis
   │
(Cache Miss)
   ▼
Azure SQL

The application logic is:

Check Redis.
If found → return the data.
If not found → read from Azure SQL.
Store the result in Redis with a TTL.
Return the response.


Interview Example

Suppose a product's price changes.

With Cache-Aside:

Update Product

↓

Database Updated

↓

Delete Product from Redis

The next read:

Redis

↓

Miss

↓

Database

↓

New Value

↓

Redis Updated

This is called cache invalidation, and it's a common way to keep cached data fresh.


Interview Answer ⭐

If asked:

"Which caching strategy would you choose for a microservices application on AKS?"

A strong answer is:

"I would typically use the Cache-Aside strategy with Azure Cache for Redis. On every read, the application first checks Redis. If the data is present (cache hit), it returns it immediately. If it's absent (cache miss), the application fetches the data from the database, stores it in Redis with an appropriate TTL, and returns it to the client. On updates, I would update the database and then invalidate or refresh the corresponding cache entry to avoid serving stale data."

