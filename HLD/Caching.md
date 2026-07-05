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