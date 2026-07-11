# **High Availability**

System continues functionaling even if some of its components fails.

High Availability means the application continues to work even if some components fail.

To make system highly available its important to understand single point of failures.
- Servers runing yout app
- Database
- Loadbalancer

Example:

Instead of:

Users
   │
   ▼
1 Server ❌

If that server crashes:

➡️ Entire application is down.

High Availability Solution

Deploy multiple instances.

          Users
             │
             ▼
      Load Balancer
       /    |     \
      ▼     ▼      ▼
   Server1 Server2 Server3

If:

Server2 ❌

Traffic automatically goes to:

Server1

and

Server3

Users usually don't notice.


## Example in AKS
Internet
    │
    ▼
Azure Front Door
    │
    ▼
API Management
    │
    ▼
AKS Service
    │
 ┌──┼──────────┐
 ▼  ▼          ▼
Pod1 Pod2     Pod3

If:

Pod2 crashes

Kubernetes Service routes traffic only to:

Pod1

Pod3

Kubernetes also creates a replacement Pod.

## How do we achieve High Availability?
1. Multiple Instances (Replicas)

Never run just one instance.

Example:

replicas: 3

If one Pod dies:

2 Pods are still serving traffic.

2. Load Balancer

Distributes requests.

Also performs health checks.

Load Balancer

↓

Healthy?

Pod1 ✅

Pod2 ❌

Pod3 ✅

Traffic goes only to healthy Pods.

3. Health Checks

In Kubernetes:

Liveness Probe → Should this Pod be restarted?
Readiness Probe → Is this Pod ready to receive traffic?

If a Pod isn't ready:

Service

↓

Don't send traffic
4. Auto Healing

Suppose:

Pod crashes

Kubernetes notices.

Pod ❌

↓

New Pod Created ✅

This happens automatically.

5. Multiple Worker Nodes

Don't put all Pods on one node.

Bad:

Node1

Pod1

Pod2

Pod3

If Node1 fails:

Everything is down.

Good:

Node1

Pod1

------------

Node2

Pod2

------------

Node3

Pod3

If one node fails, the others continue serving traffic.

6. Multi-Zone Deployment

Deploy across multiple Availability Zones.

Zone A

Pod1

------------

Zone B

Pod2

------------

Zone C

Pod3

If an entire data center (zone) goes down:

The application continues from the other zones.

7. Database High Availability

The application isn't enough.

The database must also be highly available.

Example:

Primary DB

↓

Replica

If the primary fails:

Replica can take over (depending on the database and failover setup).

8. Caching

If Redis is unavailable:

Database still works (though slower).

Similarly, if the database is under heavy read load, Redis reduces pressure and helps keep the application responsive.

Complete Architecture
Internet
     │
     ▼
Azure Front Door
     │
     ▼
API Management
     │
     ▼
Load Balancer
     │
     ▼
AKS
 ┌────┼─────────┐
 ▼    ▼         ▼
Pod1 Pod2     Pod3
 │
 ▼
Azure Cache for Redis
 │
 ▼
Azure SQL

Every layer should avoid being a single point of failure.

## High Availability vs Scalability

Many people confuse them.

High Availability	Scalability
Handles failures	Handles increased traffic
Prevents downtime	Improves capacity
Multiple instances for redundancy	More instances/resources for load
Goal: Reliability	Goal: Performance & throughput

Example:

2 Pods

Traffic is low.

One Pod crashes.

Second Pod continues.

➡️ High Availability.

2 Pods

↓

Traffic increases

↓

10 Pods

➡️ Scalability.

Interview Answer ⭐

If asked:

"How would you make your AKS application highly available?"

A strong answer is:

"I would deploy multiple Pod replicas across multiple worker nodes and, ideally, across multiple availability zones. Traffic would be distributed through a load balancer and Kubernetes Service. I'd configure readiness and liveness probes so only healthy Pods receive traffic, and Kubernetes can automatically restart failed Pods. For the infrastructure, I'd enable the Cluster Autoscaler and ensure the database and cache are configured for high availability as well."

## How deploying code in multiple region in azure cloud increase avialability?

First, let's distinguish Availability Zones from Regions, because they solve different failure scenarios.

Availability Zone (AZ): Multiple physically separate data centers within the same Azure region (e.g., East US Zone 1, Zone 2, Zone 3).
Region: A geographically separate location (e.g., East US, West Europe, Central India).
Single Region

Suppose your application is deployed only in Central India.

Users
   │
   ▼
Central India Region
   │
   ▼
AKS
Azure SQL
Redis

If the entire region has a major outage (power, networking, natural disaster):

❌ Your application becomes unavailable.

Multi-Region Deployment

Now deploy the application in two regions.

                 Users
                    │
                    ▼
           Azure Front Door
             /           \
            ▼             ▼
   Central India     South India
        AKS               AKS
      Redis             Redis

Azure Front Door routes users to the healthiest region.


What about the database?

Your application alone isn't enough.

If both AKS clusters connect to one database in one region, that database becomes a single point of failure.

A production architecture typically uses geo-replication or another disaster recovery mechanism.

Example:

Region A

AKS

↓

Azure SQL Primary

↓

Geo-Replication

↓

Azure SQL Secondary

↓

AKS

Region B

If Region A fails, the application in Region B can use the secondary database (depending on your failover strategy).


Complete Multi-Region Architecture
                    Users
                       │
                       ▼
              Azure Front Door
                 /           \
                ▼             ▼
        Region A          Region B
          AKS               AKS
           │                 │
           ▼                 ▼
        Redis             Redis
           │                 │
           ▼                 ▼
      SQL Primary  ⇄  SQL Secondary



## Availability Zones vs Regions
Availability Zone	                    Region
Protects against a data center failure	Protects against an entire regional outage
Same geographic region	                Different geographic locations
Lower latency between zones	            Higher latency between regions
Common for high availability	        Common for disaster recovery and global resilience


Interview Answer ⭐

If the interviewer asks:

"How does deploying to multiple Azure regions improve availability?"

A strong answer is:

"Deploying the application in multiple Azure regions removes the region as a single point of failure. If one region becomes unavailable due to a major outage, Azure Front Door can detect the unhealthy region through health probes and automatically route traffic to another healthy region. To achieve true high availability, the supporting services like the database and cache should also have an appropriate cross-region replication or failover strategy."


Think of it in layers:

Availability Zones (AZs) protect you from data center failures within a region.
Multiple Regions protect you from an entire regional outage.


Single Region with Multiple AZs

Suppose your AKS cluster is in Central India.

Central India Region

 ┌───────────────┐
 │ AZ-1          │
 │ Node 1        │
 │ Pod A         │
 └───────────────┘

 ┌───────────────┐
 │ AZ-2          │
 │ Node 2        │
 │ Pod B         │
 └───────────────┘

 ┌───────────────┐
 │ AZ-3          │
 │ Node 3        │
 │ Pod C         │
 └───────────────┘

If AZ-2 goes down:

AZ-1 ✅

AZ-2 ❌

AZ-3 ✅

Pods in AZ-1 and AZ-3 continue serving traffic.

Kubernetes can recreate Pods on the healthy nodes if needed.


How does AKS achieve this?

When creating an AKS cluster, you can create a node pool that spans multiple Availability Zones.

Example:

AKS Node Pool

Node 1 → AZ-1

Node 2 → AZ-2

Node 3 → AZ-3

Then Kubernetes schedules Pods across these nodes (and you can further influence placement using topology spread constraints or anti-affinity).


Region vs Availability Zone
Azure

Central India Region
   │
   ├── AZ-1
   ├── AZ-2
   └── AZ-3

South India Region
   │
   ├── AZ-1
   ├── AZ-2
   └── AZ-3

So:

Multiple AZs = Same region, different data centers.
Multiple Regions = Different geographic locations.



Best Practice

A common production setup is:

Users
   │
   ▼
Azure Front Door
   │
   ▼
Central India Region
   │
AKS Cluster
   │
Nodes in AZ-1, AZ-2, AZ-3

This protects against a single data center failure.

For even higher resilience:

Azure Front Door
      │
 ┌────┴────┐
 ▼         ▼
Central India    South India
(AZ-1/2/3)       (AZ-1/2/3)

This protects against both:

✅ Data center (AZ) failures
✅ Entire region failures

Interview Answer ⭐

If asked:

"Can we deploy across multiple Availability Zones within the same Azure region?"

A strong answer is:

"Yes. In AKS, we can configure node pools to span multiple Availability Zones within a region. Kubernetes schedules Pods across nodes in different zones, so if one availability zone fails, Pods running in the other zones continue serving traffic. This improves high availability within the region. For protection against a complete regional outage, we would deploy to multiple Azure regions and use a global traffic manager such as Azure Front Door."

## High AVailibity vs fault tolerance


High Availability (HA)

Goal: Minimize downtime.

If one component fails, the application remains available by failing over to another healthy instance.

Example:

Users
   │
   ▼
Load Balancer
   │
 ┌─┴─────────┐
 ▼           ▼
Server1    Server2

If:

Server1 ❌

The Load Balancer routes traffic to:

Server2 ✅

Users may experience a very brief interruption (milliseconds or seconds), but the service remains available.

Fault Tolerance (FT)

Goal: Continue operating without interruption, even when a component fails.

Example:

Users
   │
   ▼
Server1  +  Server2
(Both actively processing)

If:

Server1 ❌

Server2 is already processing and continues immediately.

Ideally, users notice no interruption.


Fault tolerance system is expensive than high availability system.

Comparison
High Availability	                Fault Tolerance
Goal is minimum downtime	        Goal is zero or near-zero interruption
Uses redundancy	                    Uses redundancy with simultaneous operation
Failover after failure	            Continues during failure
Small interruption is acceptable	No noticeable interruption
Lower cost	                        Higher cost


## "If a Pod crashes, Kubernetes still needs to detect it, stop sending traffic, and maybe create another Pod. That takes time."

Correct. That is High Availability, not true Fault Tolerance.

High Availability (AKS)

Suppose you have:

Load Balancer
      │
 ┌────┴────┐
 ▼         ▼
Pod A    Pod B

Pod A crashes.

What happens?

Pod A crashes

↓

Readiness/Liveness probe fails

↓

Service stops routing traffic

↓

Load Balancer sends traffic only to Pod B

↓

Kubernetes creates a new Pod A

There is a small delay (milliseconds to a few seconds depending on health check intervals and configuration).

This is High Availability.

Can Kubernetes be Fault Tolerant?

Not in the strict sense.

Kubernetes provides high availability with fast recovery, but not true zero-interruption fault tolerance.

What is True Fault Tolerance?

Imagine two systems processing the same request simultaneously.

            Client
               │
         ┌─────┴─────┐
         ▼           ▼
     Server A    Server B

Both servers are active.

If Server A dies:

Server A ❌

↓

Server B is already processing

No failover is needed because Server B never stopped working.

This is called active-active redundancy.


For your interviews

If asked:

"Is Kubernetes fault tolerant?"

A good answer is:

"Kubernetes is primarily designed for high availability, not strict fault tolerance. It minimizes downtime by running multiple replicas, performing health checks, and automatically replacing failed Pods. There may still be a brief failover period, so it's best described as a highly available platform rather than a truly fault-tolerant one."

True fault tolerance often means another system is already running and capable of continuing immediately, without waiting for failover or restart.

Fault Tolerance
             Client
                │
        ┌───────┴────────┐
        ▼                ▼
   System A         System B
   (Active)         (Also Active)

If System A fails:

System B keeps going immediately.

No need to start anything or wait for failover.



## This is a production-ready High Availability architecture for an Azure microservices application using AKS.

                           Internet Users
                                 │
                                 ▼
                    Azure Front Door (Global LB)
                    (WAF + SSL + Health Checks)
                                 │
                 ┌───────────────┴───────────────┐
                 │                               │
                 ▼                               ▼
      Central India Region             South India Region (DR)
      (Primary Region)                 (Optional Disaster Recovery)

                 │
                 ▼
               API Management
      (Authentication, Rate Limiting,
      Routing, Logging, Policies)
                 │
                 ▼
        Azure Load Balancer (AKS Service)
                 │
        ┌────────┴────────┐
        ▼                 ▼
      AZ-1              AZ-2              AZ-3
   ┌────────┐        ┌────────┐       ┌────────┐
   │ Node 1 │        │ Node 2 │       │ Node 3 │
   │        │        │        │       │        │
   │Pod A   │        │Pod B   │       │Pod C   │
   │UserSvc │        │UserSvc │       │UserSvc │
   │        │        │        │       │        │
   │Pod D   │        │Pod E   │       │Pod F   │
   │OrderSvc│        │OrderSvc│       │OrderSvc│
   └────────┘        └────────┘       └────────┘
          \              |               /
           \             |              /
            └────────────┼─────────────┘
                         │
                         ▼
               Azure Cache for Redis
                         │
                         ▼
           Azure SQL Database (Primary)
                         │
                  Geo-Replication
                         │
                         ▼
          Azure SQL Secondary (Another Region)



# Distributed System:


A distributed system is a collection of multiple independent services that work together and appear as a single system to the user.

Centralized system is a system that uses client server architecture where clients are directly connected to a single server i.e. it has single point of failure.
To scale is you simply add more cpu & memory to this single server basically vertial scaling.



Example

Suppose you open Amazon.

You simply see:

amazon.com

But behind the scenes:

                 Users
                    │
                    ▼
            Load Balancer
                    │
      ┌─────────────┼─────────────┐
      ▼             ▼             ▼
 Product Service Order Service User Service
      │             │             │
      ▼             ▼             ▼
  Product DB     Order DB     User DB

Although there are many services and databases, it behaves like one application.

This is a distributed system.

Another Example

WhatsApp

You

↓

WebSocket Server 1

↓

Redis/Kafka

↓

WebSocket Server 2

↓

Friend

You think you're chatting with one app.


Actually:

Multiple servers
Multiple databases
Message broker
Cache
Load balancers

Everything works together.


Why Distributed Systems?

Imagine one server.

Users

↓

Server

↓

Database

If:

10 million users arrive
Server crashes

Everything stops.


Instead:

Users

↓

Load Balancer

↓

Server1
Server2
Server3
Server4

Now:

More users supported
Better availability
Better performance

Benefits

✅ Scalability

Add more servers.

✅ High Availability

One server fails.

Others continue.

✅ Fault Isolation

Payment crashes.

Movies still work.

✅ Independent Deployment

Deploy only:

Recommendation Service

without touching others.


Challenges

❌ Network latency

Services communicate over the network.

❌ Data consistency

Keeping multiple databases synchronized can be difficult.

❌ Debugging

A single user request may pass through many services.

❌ Monitoring

Need centralized logging and tracing.


Example Using AKS
                 Users
                    │
            Azure Front Door
                    │
                    ▼
         Azure API Management
                    │
                    ▼
                 AKS
    ┌──────────┬──────────┬──────────┐
    ▼          ▼          ▼
 Product    Order      User Pods
 Service    Service    Service
    │          │          │
    ▼          ▼          ▼
 Redis      SQL DB    SQL DB

This entire architecture is a distributed system because it consists of multiple independently running components working together.


Interview Definition ⭐

If asked:

"What is a distributed system?"

A concise answer is:

"A distributed system is a collection of independent computers or services that communicate over a network and work together to provide a single application to users. It improves scalability, availability, and fault tolerance by distributing work across multiple machines."


Easy way to remember
Monolith
One Application

↓

One Server
Distributed System
One Application

↓

Many Servers

↓

Many Services

↓

Many Databases


Monolith

Suppose your application has:

Application

├── Product
├── Orders
├── Users
├── Payments

All modules run together.

Now imagine:

Product page gets 100,000 requests/minute
Orders get 5,000 requests/minute
Users get 2,000 requests/minute

If Product is under heavy load:

You must scale the entire application.

Before

1 Monolith Server

↓

Traffic ↑

↓

Need 5 Servers

Each server has:

Product
Order
User
Payment

Even though only Product needed more capacity.

Microservices (Distributed System)

Now split the application:

Product Service

Order Service

User Service

Payment Service

Each service runs independently.

Traffic:

Product Service

100k req/min

----------------

Order Service

5k req/min

----------------

User Service

2k req/min

Only Product Service is busy.

Scale only Product Service
Product Service

Pod1
Pod2
Pod3
Pod4
Pod5
Pod6

--------------

Order Service

Pod1

--------------

User Service

Pod1

--------------

Payment Service

Pod1

Only Product Service gets more replicas.

This is independent scaling.


In AKS

Suppose Product Deployment has:

replicas: 2

Traffic increases.

HPA changes it to:

replicas: 10

Only Product Pods increase.

Order Pods remain:

Order

2 Pods

No unnecessary scaling.


Monolith → scale the whole application.
Distributed system (microservices) → scale only the services that need it. This is one of the biggest reasons large-scale systems adopt a distributed architecture.

