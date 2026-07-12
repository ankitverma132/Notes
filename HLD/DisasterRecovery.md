# **Disaster Recovery**

What is Disaster Recovery (DR)?

Disaster Recovery means:

How does the application continue working if an entire data center, region, or cloud zone fails?

Example:

Fire in a data center
Power outage
Flood
Network failure
Entire Azure region goes down

Your application should still be available.

## Active-Active Architecture

In an Active-Active setup, both regions are live and serving user traffic at the same time.

Example:

                 Users
                    │
          Azure Front Door
                    │
        ┌───────────┴───────────┐
        ▼                       ▼
   East US Region          West Europe Region
      (Active)                (Active)

Both regions are handling requests.

Traffic Example

Suppose you have 100,000 users.

100,000 Users

↓

Front Door

↓

East US → 50,000 users

West Europe → 50,000 users

Both regions are working.

If one region fails

Suppose East US goes down.

East US ❌

Now:

Users

↓

Front Door

↓

West Europe ✅

All traffic is routed to West Europe.

Users may notice a slight delay, but the application remains available.

What needs to be replicated?

Both regions need similar infrastructure.

Region 1

AKS
Redis
Database

------------------

Region 2

AKS
Redis
Database

Data must also stay synchronized.

Advantages

✅ Very high availability

✅ Fast disaster recovery (almost immediate)

✅ Better global performance (users connect to the nearest region)

Disadvantages

❌ More expensive (you're running two complete environments)

❌ More complex data synchronization

Active-Passive

Only one region is serving traffic.

               Users
                  │
           Azure Front Door
                  │
                  ▼
           Region 1 (Active)

------------------------------

           Region 2 (Passive)

Region 2 is on standby.

Disaster

If Region 1 fails:

Region 1 ❌

↓

Front Door

↓

Region 2 becomes Active

There may be some failover time depending on the setup.

Active-Active vs Active-Passive
Active-Active	Active-Passive
Both regions serve traffic	Only one region serves traffic
Faster failover	Slower failover
Higher cost	Lower cost
Better global performance	Simpler architecture
More complex data synchronization	Simpler synchronization
Azure Example

Suppose your application is:

Users
   │
Azure Front Door
   │
 ┌─┴───────────────┐
 ▼                 ▼
East US         Central India

Both regions have:

AKS

↓

Azure Cache for Redis

↓

Azure SQL Database

Front Door routes users to the healthiest and/or nearest region.

If East US goes down:

Users

↓

Front Door

↓

Central India

Traffic continues.

Interview Answer ⭐

If asked:

"What is an Active-Active disaster recovery solution?"

A good answer is:

"In an Active-Active disaster recovery architecture, the application is deployed in two or more regions, and all regions actively serve production traffic. A global load balancer such as Azure Front Door or AWS Route 53 distributes requests between them. If one region becomes unavailable, traffic is automatically routed to the remaining healthy region(s), providing high availability with minimal downtime. The trade-off is higher infrastructure cost and the need to keep data synchronized across regions."


## What is RPO?

RPO = Recovery Point Objective

It answers the question:

"How much data can we afford to lose after a disaster?"

It is measured in time.

Example 1

Suppose your database is backed up every 1 hour.

Timeline:

10:00  Backup

11:00  Backup

11:30  Disaster 💥

The latest backup is from 11:00.

So you lose data between:

11:00 → 11:30

You lost 30 minutes of data.

RPO = 1 hour (because backups occur every hour, you could lose up to one hour of data).

Example 2

If your data is replicated continuously:

Database A  <------>  Database B

A disaster occurs at:

11:30

The replica is already up to date.

Data loss:

0 minutes

RPO ≈ 0

Real-world Examples
Banking System

If a bank loses:

₹10,000 transaction

That's unacceptable.

Desired:

RPO = 0

No transaction loss.

Social Media

Suppose a "like" from the last minute is lost.

Usually acceptable.

Example:

RPO = 5 minutes

Lower RPO means...

To achieve a lower RPO, you typically need:

Continuous replication
More frequent backups
Better infrastructure

But it costs more.

## RPO vs RTO

People often confuse these.

RPO

How much data can I lose?

Example:

30 minutes of data lost
RTO (Recovery Time Objective)

How long can the application be down?

Example:

App back online in 10 minutes

Example

Disaster:

12:00

↓

Application crashes

Recovered:

12:10

Downtime:

10 minutes

RTO = 10 minutes


Interview Summary
Term	Meaning
RPO	    Maximum acceptable data loss after a disaster
RTO	    Maximum acceptable downtime before the service is restored


Interview Answer ⭐

If asked:

"What is RPO?"

A good answer is:

"Recovery Point Objective (RPO) is the maximum amount of data an organization is willing to lose after a disaster. It is measured in time. For example, if backups are taken every hour, the RPO is up to one hour because, in the worst case, up to one hour of data could be lost."

Quick memory trick
RPO → Point = How far back does my data roll back?
RTO → Time = How long until my application is running again? Amount of time application can be down during disaster.


## Disaster recovery strategies:

Here are 4 main DR strategies, ranging from cheapest to most expensive.

1. Backup & Restore

The simplest strategy.

Architecture:

Production

↓

Backup

↓

Azure Blob Storage

If disaster happens:

Restore Backup

↓

Deploy Application

↓

Application Running
Example
Daily database backup
AKS manifests stored in Git
Container images in Azure Container Registry

If the region fails:

Create a new AKS cluster
Restore database
Deploy application
Pros
✅ Cheapest
✅ Simple
Cons
❌ Long recovery time
❌ Data loss possible

Typical:

RTO = Hours
RPO = Hours

Imagine your application
Users
   │
Load Balancer
   │
AKS
   │
Azure SQL Database

Everything is running in Region A.

Every night you take a backup

For example:

Azure SQL
      │
      ▼
Backup stored in Azure Storage

Also:

Your AKS deployment YAML is stored in Git.
Your Docker images are stored in Azure Container Registry.

So you have backups of everything needed to rebuild the application.

Disaster happens

Suppose Region A is completely unavailable.

Region A

Load Balancer ❌

AKS ❌

Azure SQL ❌

Nothing is running anymore.

What do you do?

You build everything again in another region.

Step 1

Create new AKS cluster

↓

Step 2

Deploy application

↓

Step 3

Restore database from backup

↓

Step 4

Create Load Balancer

↓

Step 5

Switch DNS/Front Door

Now the application works again.

Why is it called "Backup & Restore"?

Because nothing is running in the secondary region before the disaster.

You only have:

Database backup
Storage backup
Source code
Kubernetes YAML
Container images

That's it.

No AKS.

No Pods.

No Load Balancer.

Visual
Before Disaster
Region A

AKS ✅
SQL ✅
Load Balancer ✅

Region B

Nothing

Only Backups 📦
After Disaster
Restore Backup

↓

Create AKS

↓

Deploy Pods

↓

Restore Database

↓

Application Live
Why is it cheap?

Because you're not paying for another AKS cluster or running VMs.

You're only paying to store backups.

Why is it slow?

Imagine your production system has:

20 AKS nodes
50 microservices
500 GB database

After a disaster you must:

Create the AKS cluster
Download container images
Deploy all services
Restore the database
Configure networking
Verify everything

This can take hours.


2. Pilot Light

Keep only the critical infrastructure running.

Primary Region

AKS
Redis
SQL

-------------------

Secondary Region

Database Replica ✅

Everything else OFF

If disaster occurs:

Start AKS

↓

Deploy Pods

↓

Switch Traffic
Pros
Faster than Backup & Restore
Lower cost than Active-Passive
Cons
Some startup delay

Typical:

RTO = Minutes
RPO = Very low (depends on replication)


3. Active-Passive (Warm Standby)

A smaller version of production is already running.

Region A

AKS (10 Pods)

---------------

Region B

AKS (2 Pods)

The standby region is live but handles little or no production traffic.

If Region A fails:

Increase Pods

2

↓

10

Switch traffic.

Pros
Fast recovery
Easier than Active-Active
Cons
Paying for two environments

Typical:

RTO = Minutes
RPO = Near zero (with replication)

4. Active-Active

Both regions are fully operational.

Users

↓

Front Door

↓

Region A

Region B

Both serve users simultaneously.

If Region A fails:

Front Door

↓

Route Everything

↓

Region B

Users usually notice little or no downtime.

Pros
Highest availability
Fastest recovery
Cons
Most expensive
Most complex

Typical:

RTO ≈ 0
RPO ≈ 0 (if data replication is designed appropriately)

Comparison
Strategy	                    Cost	    Recovery Time (RTO)	    Data Loss (RPO)
Backup & Restore	            ⭐ Lowest	Hours	                Hours
Pilot Light         	        ⭐⭐	        Minutes to Hours	    Low
Active-Passive (Warm Standby)	⭐⭐⭐	        Minutes	Near Zero
Active-Active	                ⭐⭐⭐⭐        Highest	Near Zero	Near Zero


### Azure Example

Suppose your application is:

Users
   │
Azure Front Door
   │
AKS
   │
Redis
   │
Azure SQL
Backup & Restore
Database backups
AKS YAML in Git
Redis rebuilt if needed

Pilot Light
Primary

AKS

SQL

---------------

Secondary

SQL Replica
Active-Passive
Primary

AKS (10 Pods)

---------------

Secondary

AKS (2 Pods)
Active-Active
Users

↓

Azure Front Door

↓

East US AKS

West Europe AKS

Both regions actively serve traffic.

