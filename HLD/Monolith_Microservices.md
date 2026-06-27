# **Monolith vs MicroService**

Monolithic Architecture

A monolith is a single application where all features are part of one codebase and are deployed together.

For example, imagine you're building an e-commerce website.

Everything is inside one application:

E-commerce App
│
├── User Login
├── Product Catalog
├── Cart
├── Orders
├── Payments
├── Notifications
└── Admin Panel

When a user requests:

Browser
    │
    ▼
Single Server
    │
    ├── Login Module
    ├── Product Module
    ├── Order Module
    ├── Payment Module
    └── Notification Module

Even though they're different modules, they're all part of one deployed application.

Advantages
✅ Easy to develop initially.
✅ Simple deployment (one application).
✅ Easier debugging.
✅ Good for startups and small products.
Disadvantages
Scaling is a challenge.
All or nothing deployment.
Long release cycle.
Slow to react to customer demand
Difficult to test.

Suppose only the payment system gets heavy traffic during a sale.
You cannot scale just payments.
You have to scale the entire application.

Microservices Architecture

Now imagine every feature is its own independent service.

Browser
    │
    ▼
API Gateway / Load Balancer
    │
    ├── User Service
    ├── Product Service
    ├── Order Service
    ├── Payment Service
    └── Notification Service

Each service:

has its own code
can be deployed independently
can be scaled independently

For example:

Order Service
      │
      ▼
Order Database

Payment Service
      │
      ▼
Payment Database

Example
Suppose Amazon is having a big sale.

Traffic:

Login        -> Normal
Products     -> High
Orders       -> Very High
Payments     -> Extremely High
Notifications-> Medium

With a monolith:

Need 10 more servers

↓

Entire application copied 10 times

With microservices:

Need more Payments

↓

Only Payment Service gets 10 more servers

This is much more efficient.



Other Pros:
All services can writtent in different languages.

Characteristics of Microservices:
Independent Scaling, Testing, Deployment, Monitoring.


## Handling traffic with kubernetes:

Let's say you have these microservices:

User Service
Product Service
Order Service
Payment Service
Notification Service

During a sale, the traffic looks like this:

User           → 100 requests/sec
Product        → 2,000 requests/sec
Order          → 1,500 requests/sec
Payment        → 300 requests/sec
Notification   → 50 requests/sec

You don't want every service to have the same number of servers.

Without Kubernetes

You would manually add more servers for the Product and Order services.

Product Service
Server 1
Server 2
Server 3
Server 4
Server 5

Order Service
Server 1
Server 2
Server 3

Notification
Server 1

This is tedious and doesn't adapt automatically if traffic changes.

With Kubernetes

In Kubernetes, your application runs in Pods (the smallest deployable unit, usually containing one container).

You tell Kubernetes how many pod replicas each service should have.

Initially:

Product Service
2 Pods

Order Service
2 Pods

Notification Service
1 Pod

If Product Service starts receiving heavy traffic:

Product Service

Pod 1
Pod 2
Pod 3
Pod 4
Pod 5
Pod 6
Pod 7
Pod 8

Meanwhile:

Notification Service

Pod 1

It stays at one pod because it doesn't need more capacity.

How does traffic reach the right pod?

Kubernetes provides a Service, which acts like a stable endpoint and load balances requests across the available pods.

Product Service
        │
        ▼
 Kubernetes Service
        │
 ┌──────┼──────┐
 ▼      ▼      ▼
Pod1   Pod2   Pod3

If Kubernetes adds more pods:

        │
        ▼
 Kubernetes Service
        │
 ┌──┬──┬──┬──┬──┬──┐
 ▼  ▼  ▼  ▼  ▼  ▼
P1 P2 P3 P4 P5 P6

Traffic is automatically distributed among them.

Automatic Scaling

Kubernetes can automatically increase or decrease the number of pods using a Horizontal Pod Autoscaler (HPA).

Example policy:

If CPU usage > 70%

Increase replicas

Traffic grows:

2 Pods

↓

CPU 80%

↓

4 Pods

↓

CPU still 85%

↓

8 Pods

Later at night:
CPU 10%

↓

Scale back to

2 Pods

So you only use the resources you actually need.


Different services can scale independently

For example:

Product Service       → 12 Pods
Order Service         → 8 Pods
Payment Service       → 3 Pods
Notification Service  → 1 Pod
User Service          → 2 Pods

Each service has its own scaling policy.

One important point

Kubernetes doesn't decide on its own which service should get more pods. You (or your DevOps team) configure scaling rules, such as:

CPU utilization (e.g., keep average CPU around 70%)
Memory usage
Custom metrics (e.g., requests per second, queue length)

Based on those rules, Kubernetes automatically adds or removes pod replicas.

he number of replicas is defined in the Deployment manifest in Kubernetes.

Here's a simple example:

apiVersion: apps/v1
kind: Deployment

metadata:
  name: product-service

spec:
  replicas: 3   # 👈 Number of Pods

  selector:
    matchLabels:
      app: product-service

  template:
    metadata:
      labels:
        app: product-service

    spec:
      containers:
      - name: product-service
        image: my-product-service:v1

The important line is:

spec:
  replicas: 3

This tells Kubernetes:

"Always keep 3 Pods of the Product Service running."

So Kubernetes creates:

Product Service

Pod 1
Pod 2
Pod 3
What if one Pod crashes?

Suppose:

Pod 2 ❌

Now only 2 Pods are running.

Kubernetes continuously watches the cluster and notices that the desired state (replicas: 3) is no longer met.

It immediately starts a new Pod:

Pod 1
Pod 3
Pod 4   ← New Pod

You don't have to restart it manually.


How do we increase replicas?
Method 1: Change the YAML

Update:

replicas: 5

Then apply it:

kubectl apply -f deployment.yaml

Kubernetes creates two more Pods.

Method 2: Using kubectl

You can scale without editing the file:

kubectl scale deployment product-service --replicas=10

Now Kubernetes keeps 10 Pods running.

Method 3: Horizontal Pod Autoscaler (recommended)

Instead of a fixed number:

replicas: 3

You configure an autoscaler, for example:

minReplicas: 2
maxReplicas: 20
targetCPUUtilizationPercentage: 70

Now Kubernetes automatically adjusts:

Low traffic
↓

2 Pods

----------------

Medium traffic
↓

5 Pods

----------------

Heavy traffic
↓

15 Pods

----------------

Traffic decreases
↓

3 Pods

You don't have to manually change the replica count.

### Interview tip

Remember the relationship between these Kubernetes objects:

Deployment
    │
    │ defines replicas
    ▼
ReplicaSet
    │
    │ ensures the desired number of Pods exist
    ▼
Pods
Deployment → You define the desired state (image, replicas, update strategy).
ReplicaSet → Maintains the specified number of Pod replicas.
Pods → Run your application containers.