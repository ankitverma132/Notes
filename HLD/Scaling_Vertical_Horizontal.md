# **Scaling Vertical & Horizontal**

Vertical Scaling (Scale Up)

Increase the power of a single machine.

For example:

Server

CPU : 4 Core
RAM : 8 GB

Upgrade it to:

Server

CPU : 16 Core
RAM : 64 GB

Same server, but more powerful.

Diagram
Before

        Server
     4 CPU
     8 GB RAM

↓

After

        Server
     16 CPU
     64 GB RAM
Advantages
✅ Easy to implement
✅ No application changes
✅ Good for monolithic applications
Disadvantages
❌ There's a hardware limit.
❌ Usually requires downtime to upgrade.
❌ Expensive high-end machines.
❌ Single point of failure.
If traffic reduces still application runs on bigger server. Means, still you have to pay for bigger server.
Scaling up and down take longer.
Chance of missing transaction during cutover.

If the server crashes:

Users

↓

Server ❌

Everything is down.

Horizontal Scaling (Scale Out)

Instead of buying a bigger server, add more servers.

Example:

1 Server

becomes

5 Servers
Diagram
             Load Balancer
           /      |      \
          ▼       ▼       ▼
      Server1  Server2  Server3

Traffic is distributed among them.

Advantages
✅ Almost unlimited scaling.
✅ High availability.
✅ Fault tolerance.
✅ Easier to handle traffic spikes.

If one server crashes:

Load Balancer
     │
 ┌───┴─────┐
 ▼         ▼
Server1  Server2
   ❌        ✅

Users are still served by Server 2.

Disadvantages
❌ More complex.
❌ Requires a load balancer.
❌ Application should ideally be stateless.
❌ Data synchronization becomes more challenging.

Example

Suppose your app gets:

500 requests/sec
Vertical Scaling
Old Server

4 CPU

↓

Upgrade

16 CPU

Still one server.


Horizontal Scaling
Load Balancer
    │
 ┌──┼───────┐
 ▼  ▼       ▼
S1 S2      S3

Each server handles part of the traffic.
You just add up new Servers. Very low chance of missing transaction.
Scaling up and down is faster. Massevily scalable
Cost effective. Simply remove new server if traffic goes down.


Which one do cloud providers use?

Mostly Horizontal Scaling.

For example, in Kubernetes:

Product Service

Pod1
Pod2
Pod3
Pod4

When traffic increases:

Product Service

Pod1
Pod2
Pod3
Pod4
Pod5
Pod6
Pod7
Pod8

Kubernetes simply creates more Pods.


Interview Comparison
Vertical Scaling	        Horizontal Scaling
Scale Up	                Scale Out
Increase CPU/RAM	        Add more servers
One machine	                Multiple machines
Limited by hardware	        Almost unlimited
Simpler	                    More complex
Single point of failure	    Better fault tolerance
Usually downtime to upgrade	Can often scale without downtime


## Scaling in VM, Serverless & Containers
1. Virtual Machines (VMs)

Example:

Azure Virtual Machine
Amazon EC2
Google Compute Engine
Architecture
            Load Balancer
          /      |      \
         ▼       ▼       ▼
      VM1      VM2      VM3

Each VM has:

Operating System
Runtime
Application
Scaling

Horizontal scaling:

Before

VM1
VM2

↓

Traffic Increases

↓

VM1
VM2
VM3
VM4
VM5

Vertical scaling:

VM

2 CPU
4 GB RAM

↓

8 CPU
32 GB RAM
Pros
Full control
Supports any software
Good for legacy applications
Cons
Slow startup (30 seconds–a few minutes)
OS management
Patching
Higher cost

2. Containers
Only Scales horizontly.
Example:

Kubernetes (AKS, EKS, GKE)
Docker

A container shares the host OS kernel, making it much lighter than a VM.

Worker Node

---------------------------------
Container 1
Container 2
Container 3
---------------------------------
Linux Host
Scaling

Kubernetes simply creates more Pods.

Product Service

Pod1
Pod2

↓

High Traffic

↓

Pod1
Pod2
Pod3
Pod4
Pod5

Using a Horizontal Pod Autoscaler (HPA):

CPU > 70%
Increase replicas
Pros
Fast startup (seconds)
Efficient resource usage
Great for microservices
Portable across environments
Cons
Kubernetes adds operational complexity
You still manage the cluster (unless using managed offerings)
3. Serverless

Examples:

AWS Lambda
Azure Functions
Google Cloud Functions

No servers or containers to manage directly.

User

↓

Function
Scaling

Requests arrive:

100 Requests

↓

100 Function Instances

Then traffic drops:

0 Requests

↓

0 Function Instances

This is called scale to zero.

Pros
No infrastructure management
Automatic scaling
Pay only for execution time
Scale to zero when idle

Cons
Cold starts (depending on platform/configuration)
Execution time limits (platform-dependent)
Less control over the runtime environment

Comparison
Feature	                VM	                Container	                                Serverless
Startup Time	        Slow	            Fast	                                    Very Fast (cold starts possible)
OS Management	        You manage	        Shared host OS	                            Provider manages
Scaling	                VM Autoscaling	    Pod Autoscaling	                            Automatic
Scale to Zero	        ❌	        Possible with additional tools (e.g., KEDA)	        ✅
Cost	                Highest	            Medium	                                    Lowest for sporadic workloads
Control	                Highest	            High	                                    Lowest
Best For	            Legacy apps	        Microservices	                            Event-driven apps

Real Example

Suppose your application gets:

100,000 requests
VM
Load Balancer

↓

VM1
VM2
VM3
VM4

A VM Auto Scaling Group launches additional VMs, which can take some time.

Kubernetes
Load Balancer

↓

Pod1
Pod2
Pod3
Pod4
Pod5
Pod6

The HPA creates more Pods, usually much faster than launching VMs.

Serverless
100,000 Requests

↓

100,000 Function Executions

The cloud provider automatically provisions execution environments as needed.

Typical Interview Question

Which one scales the fastest?

Generally:

Serverless > Containers > VMs

Why?

Serverless: The cloud provider automatically provisions execution environments.
Containers: Starting a new container or Pod is usually much faster than booting a full VM.
VMs: Booting a new operating system takes the longest.

Interview one-liner ⭐

VMs scale by adding or resizing virtual machines, containers scale by adding more container instances (Pods), and serverless platforms automatically create function instances on demand—often even scaling down to zero when there's no traffic.




## How can we make our app more scalable for big traffic day?
If we are deploying our application on Kubernetes, we can improve scalability using the following techniques:

1. Deploy Multiple Pod Replicas

Instead of running a single instance of the application, deploy multiple replicas.

Before

App
 │
Pod 1

--------------------

After

App
 │
├── Pod 1
├── Pod 2
├── Pod 3
├── Pod 4

This allows multiple requests to be processed in parallel.

2. Use a Kubernetes Service

A Kubernetes Service acts as an internal load balancer.

Client
   │
   ▼
Kubernetes Service
   │
 ┌─┼────────┐
 ▼ ▼        ▼
P1 P2      P3

Incoming requests are distributed across healthy Pods.

3. Horizontal Pod Autoscaler (HPA)

Configure HPA to automatically increase or decrease the number of Pods.

Example:

Min Pods : 3
Max Pods : 20

Target CPU : 70%

When traffic increases:

3 Pods

↓

CPU 85%

↓

6 Pods

↓

CPU 90%

↓

12 Pods

When traffic decreases, Kubernetes scales back down.

4. Cluster Autoscaler

What if there isn't enough CPU or memory to schedule new Pods?

Example:

Node 1
Pod
Pod
Pod

Node 2
Pod
Pod
Pod

A new Pod cannot be scheduled because all nodes are full.

The Cluster Autoscaler automatically adds a new worker node.

Node 1
Node 2
Node 3  ← New Node

Now Kubernetes schedules the pending Pods on the new node.

5. Liveness and Readiness Probes

Ensure traffic is sent only to healthy Pods.

Pod 1 ✅
Pod 2 ✅
Pod 3 ❌

Kubernetes stops routing requests to Pod 3 until it becomes healthy (or restarts it if needed).

6. Resource Requests and Limits

Configure CPU and memory requests/limits.

Example:

requests:
  cpu: 500m
  memory: 512Mi

limits:
  cpu: 1
  memory: 1Gi

This prevents one Pod from consuming excessive resources and affecting others.

7. Stateless Application

Keep application instances stateless.

Store sessions in:

Redis
Database
JWT

Instead of inside Pod memory.

This allows any Pod to serve any request.

Complete Architecture
                 Users
                   │
                   ▼
          Load Balancer / Ingress
                   │
                   ▼
          Kubernetes Service
                   │
         ┌─────────┼─────────┐
         ▼         ▼         ▼
      Pod 1     Pod 2     Pod 3
         │
         ▼
   Horizontal Pod Autoscaler
         │
         ▼
   More Pods when traffic increases

If cluster capacity is exhausted:

Cluster Autoscaler
        │
        ▼
   Add New Worker Node
Interview Answer (2-minute version)

"To handle a high-traffic day, I would deploy the application on Kubernetes with multiple Pod replicas behind a Kubernetes Service for load balancing. I'd configure a Horizontal Pod Autoscaler to scale Pods based on CPU, memory, or custom metrics. If the cluster runs out of capacity, the Cluster Autoscaler would automatically add worker nodes. I'd also keep the application stateless so any Pod can handle any request, use readiness and liveness probes to ensure traffic goes only to healthy Pods, and configure appropriate resource requests and limits to maintain stable performance."


## Scheduled Scaling with Kubernetes.
Yes! Scheduled scaling is absolutely possible, and it's commonly used in production.

This is especially useful when you know traffic will increase at a predictable time.

Example

Suppose you're running an e-commerce website.

Every day:

8 PM - 11 PM

traffic increases by 5x.

Instead of waiting for the HPA to detect high CPU:

8:00 PM

CPU rises

↓

HPA notices

↓

Creates Pods

↓

Pods become Ready

There is a short delay.

Scheduled Scaling

Instead:

7:45 PM

Increase replicas from

5 Pods

↓

20 Pods

Now when traffic arrives at 8 PM:

Users

↓

20 Pods already running

↓

No delay
How is it done in Kubernetes?

Kubernetes doesn't have native scheduled autoscaling in the core API.

Common approaches include:

CronJobs that run kubectl scale
External controllers like KEDA (supports cron-based scaling)
Cloud-provider features (e.g., Azure AKS, AWS EKS, Google GKE integrations)
CI/CD pipelines or automation scripts

For example:

7:45 PM

kubectl scale deployment product-service --replicas=20

11:30 PM

kubectl scale deployment product-service --replicas=5
Combining Scheduled Scaling + HPA ⭐

This is actually a common production setup.

Example:

Scheduled Scaling

7:45 PM

↓

20 Pods

↓

Traffic becomes even higher than expected

↓

HPA

↓

20 → 35 Pods

So:

Scheduled scaling prepares for known traffic spikes.
HPA handles unexpected spikes.
Real-world examples
🛒 E-commerce during Black Friday or flash sales
🎫 Ticket booking when sales open at a fixed time
📺 Streaming platforms before a major sports event
📈 Stock trading apps during market open
🏦 Banking apps around salary credit dates

Interview answer ⭐

If asked:

"Can Kubernetes perform scheduled scaling?"

A strong answer is:

"Yes. While Kubernetes' built-in Horizontal Pod Autoscaler is reactive, scheduled scaling is also possible using tools like KEDA with cron triggers, Kubernetes CronJobs, or cloud-provider automation. In production, a common pattern is to combine scheduled scaling for predictable traffic spikes with HPA for unexpected demand. This ensures capacity is available before traffic arrives while still adapting to real-time load."


## Kubernetes 2 level of Scaling(Substitute to AWS Auto scaling):
1. Horizontal Pod Autoscaler (HPA)

Scales Pods.

Product Service

2 Pods

↓

High CPU

↓

6 Pods

Based on:

CPU
Memory
Custom metrics
Requests/sec (via custom metrics)
2. Cluster Autoscaler

Scales worker nodes (VMs).

Node 1
Node 2

↓

Pods Pending

↓

Node 3 Added

This is the Kubernetes equivalent of VM scaling.

In AKS, the Cluster Autoscaler scales node pools, not individual Pods.

Traffic ↑
      │
      ▼
HPA → More Pods
      │
      ▼
No space on existing nodes
      │
      ▼
Cluster Autoscaler → Adds a new VM to the AKS node pool
      │
      ▼
Scheduler places the new Pods on that node


## Kubernetes / AKS Scaling Summary:

There are 5 major ways to scale applications on Kubernetes/AKS.

1. Horizontal Pod Autoscaler (HPA) ⭐⭐⭐⭐⭐

Scales Pods

Trigger:

CPU
Memory
Custom Metrics
Requests/sec
Queue Length
3 Pods

↓

High CPU

↓

8 Pods

Use Case:
Dynamic traffic increase.

2. Cluster Autoscaler ⭐⭐⭐⭐⭐

Scales Worker Nodes (VMs)

When:

Need 10 Pods

↓

Current Nodes Full

↓

Add New Node

In AKS:

AKS Node Pool

2 Nodes

↓

4 Nodes

Azure creates new VMs automatically.

3. Manual Scaling ⭐⭐⭐

Developer/Admin manually changes replicas.

kubectl scale deployment product --replicas=20

Useful during:

Testing
Emergency situations
Planned maintenance

4. Scheduled Scaling ⭐⭐⭐⭐

Traffic is predictable.

Example:

Every Day

8 PM Sale

↓

Increase Pods

5 → 25

Can be done using:

KEDA (Cron Trigger)
CronJobs
Azure Automation
CI/CD Pipelines

Great for:

Black Friday
Cricket match
Flash sale
Salary day

5. Vertical Pod Scaling (VPA) ⭐⭐⭐

Instead of increasing Pods:

Increase Pod resources.

Before

CPU 500m

Memory 512 MB

↓

After

CPU 2 Core

Memory 2 GB

Useful when:

Application cannot be horizontally scaled.
Legacy applications.
Some databases.

Complete Flow
                 Traffic ↑
                     │
                     ▼
         Horizontal Pod Autoscaler
                     │
          More Pods Required
                     │
          Enough Node Capacity?
              │             │
             Yes           No
              │             │
              ▼             ▼
         Schedule Pods   Cluster Autoscaler
                              │
                         Add New Node
                              │
                              ▼
                      Schedule New Pods