# **API Gateway**

An API Gateway is the single entry point for all client requests in a microservices architecture.
API gateway sits between client and a collection of backend services.
API Gateway accepts all API calls, aggregate various services to fulfill request then return an appropriate response.

Instead of clients calling each microservice directly:

Client
   │
   ├──► User Service
   ├──► Product Service
   ├──► Order Service
   └──► Payment Service

The client talks only to the API Gateway:

              Client
                 │
                 ▼
          API Gateway
        ┌────┼─────┬─────┐
        ▼    ▼     ▼     ▼
     User  Product Order Payment
    Service Service Service Service

The gateway decides which service should receive the request.

Why do we need an API Gateway?

Imagine you have 50 microservices.

Without a gateway:

Mobile app needs to know all 50 endpoints.
Web app needs to know all 50 endpoints.
If a service URL changes, every client must be updated.

With an API Gateway:

Client

https://api.myapp.com
        │
        ▼
API Gateway
        │
        ▼
Correct Service

Clients only know one URL.

What does an API Gateway do?
1. Request Routing
GET /products

↓

Product Service
POST /orders

↓

Order Service
2. Authentication & Authorization

Instead of every service validating JWTs:

Client
   │ JWT
   ▼
API Gateway
Validate Token
   │
   ▼
Order Service

The gateway can reject unauthorized requests before they reach the backend.

3. Rate Limiting

Suppose one user sends:

10000 requests/sec

The gateway can enforce a limit like:

100 requests/minute

Extra requests are rejected (often with HTTP 429).

4. Request Aggregation

Imagine a product page needs:

Product details
Reviews
Inventory

Without a gateway:

Mobile

↓

Product Service

↓

Review Service

↓

Inventory Service

Three separate API calls.

With a gateway:

Mobile

↓

API Gateway

↓

Product
Review
Inventory

↓

Combined Response

The client makes just one request.

5. Logging & Monitoring

Every request passes through the gateway, making it a convenient place to:

Log requests
Measure latency
Collect metrics
Trace requests

6. Load Balancing

Some API gateways can also distribute requests across multiple instances of a service.

7. Caching ✅

Suppose thousands of users request the same product.

Without caching:

Client
   │
   ▼
API Gateway
   │
   ▼
Product Service
   │
   ▼
Database

Every request hits the database.

With gateway caching:

Client
   │
   ▼
API Gateway Cache
   │
Cache Hit ✅

(No call to Product Service)

Only the first request reaches the backend.

Example:

GET /products/123

The response is cached for, say, 60 seconds.

The next 10,000 requests can be served directly from the gateway cache.

Benefit:

Lower latency
Reduced backend load
Fewer database queries

8. Request Validation ✅

Instead of every microservice validating input:

{
  "email": "abc",
  "password": ""
}

The API Gateway checks:

Required fields
Data types
JSON schema
Request size
Content-Type

If invalid:

400 Bad Request

The request never reaches your service.

9. Response Validation / Transformation ✅

The gateway can also inspect or modify responses.

Example:

Backend returns:

{
  "name": "Ankit",
  "password": "123",
  "email": "a@gmail.com"
}

The gateway can remove sensitive fields:

{
  "name": "Ankit",
  "email": "a@gmail.com"
}

Or transform data into a different format if required by clients.

10. Request Transformation

The gateway can modify incoming requests.

Example:

Client sends:

{
   "userName":"ankit"
}

Gateway transforms it into:

{
   "username":"ankit"
}

before forwarding it.

11. Response Transformation

Backend:

{
   "firstName":"Ankit",
   "lastName":"Verma"
}

Gateway:

{
   "fullName":"Ankit Verma"
}

Useful when supporting different clients or API versions.



## API Gateway vs Load Balancer

Feature	                            ALB	                    API Gateway
Primary purpose	                Distribute traffic	        Manage APIs
Layer	                        Layer 7 (HTTP/HTTPS)	    Layer 7 (API Management)
Load Balancing	                      ✅	                    Some gateways support it
Path-based routing	                  ✅	                        ✅
Host-based routing	                  ✅	                        ✅
Authentication	                Basic integration	         ✅ Rich authentication
Rate Limiting	                       ❌ Limited	            ✅
API Keys	                           ❌	                    ✅
Request Validation	                   ❌	                    ✅
Response Transformation	               ❌	                    ✅
Request Transformation	               ❌	                    ✅
Caching	                               ❌ (No response caching)	✅

A simple way to remember it:

Load Balancer = "Which server should handle this request?"
API Gateway = "How should this API request be processed before reaching the service?"

ALB is primarily responsible for distributing incoming network traffic across gateway instances or backend targets and handling infrastructure-level concerns like health checks.

API Gateway focuses on API management, such as authentication, rate limiting, request/response transformations, caching, validation, and routing to microservices.


Do they work together?

Yes, very often.

Internet
    │
    ▼
Application Load Balancer
    │
    ▼
API Gateway
    │
 ┌──┼─────────┐
 ▼  ▼         ▼
User Product Order

The load balancer distributes incoming traffic to gateway instances, and the API Gateway applies policies like authentication, rate limiting, and routing to the appropriate microservice.


Think of it like this
ALB = Traffic Manager 🚦

Its job is simply:

"Which backend server should receive this request?"

User
   │
   ▼
ALB
   │
 ┌─┴─────────┐
 ▼           ▼
Server1   Server2
API Gateway = API Manager 🛂

Its job is:

"Can this request enter? Should I cache it? Is the user authenticated? Which service should receive it? Should I transform the request?"

User
   │
Authenticate
Rate Limit
Validate
Cache
Transform
Route
   │
   ▼
Microservice

## Health check in kubernetes
In Kubernetes

A common architecture looks like:

Client
   │
   ▼
ALB
   │
   ▼
API Gateway
   │
   ▼
Kubernetes Service
   │
 ┌─────┴──────┐
 ▼            ▼
Pod 1      Pod 2

Health checks happen at multiple layers:

ALB → Checks whether its registered targets are healthy.
Kubernetes → Uses readiness and liveness probes to determine whether Pods should receive traffic or be restarted.
API Gateway → Focuses on API concerns rather than continuously health-checking backend instances.

## Static IP:
This is another common AWS interview question.

ALB — ❌ No Static Public IP

An Application Load Balancer (ALB) does not provide fixed public IP addresses.

Why?

AWS can:

Scale the ALB up or down.
Replace the underlying infrastructure.
Change the IP addresses behind the scenes.

That's why AWS recommends using the DNS name of the ALB, for example:

my-alb-123456.ap-south-1.elb.amazonaws.com

The DNS name stays the same even if the underlying IPs change.

NLB — ✅ Supports Static IP

A Network Load Balancer (NLB) can have static IP addresses.

You can even associate Elastic IPs (EIPs) with it.

Example:

Client
   │
203.0.113.10  ← Static IP
   │
   ▼
NLB

This is useful when:

A firewall needs to whitelist your IP.
A partner system only accepts traffic from fixed IPs.
Compliance requires static addresses.

API Gateway — ❌ No Static Public IP

AWS API Gateway exposes a hostname like:

https://abc123.execute-api.ap-south-1.amazonaws.com

It does not provide a fixed public IP address.

If you need a static IP in front of API Gateway, a common solution is:

Client
   │
Static IP
   ▼
AWS Global Accelerator
   │
   ▼
API Gateway

or use an NLB in architectures where it's appropriate.


Interview Summary
Service	    Static Public IP
ALB	        ❌ No
NLB	        ✅ Yes
API Gateway	❌ No (use DNS; static IP requires another AWS service like Global Accelerator)


## Multiple API Gateway Instance:

You have multiple API Gateway instances, the ALB's job is to distribute requests among them.

Architecture
                 Internet
                     │
                     ▼
         Application Load Balancer
          /          |           \
         ▼           ▼            ▼
 API Gateway 1  API Gateway 2  API Gateway 3
         │           │            │
         └──────┬────┴────────────┘
                ▼
         Product Service
         Order Service
         Payment Service
Step-by-step flow

A user requests:

GET /products/123
Step 1: ALB

The ALB chooses one healthy API Gateway instance.

For example:

User

↓

ALB

↓

API Gateway 2
Step 2: API Gateway

API Gateway 2 performs:

✅ Authentication
✅ Rate limiting
✅ Request validation
✅ Caching
✅ Logging

Then routes:

GET /products/123

↓

Product Service
Step 3: Product Service

Product Service processes the request and returns:

{
  "id":123,
  "name":"iPhone"
}

The response goes back through the API Gateway and ALB to the client.

Why multiple API Gateway instances?

Imagine:

100,000 requests/sec

One gateway instance won't be enough.

So you deploy:

Gateway 1
Gateway 2
Gateway 3
Gateway 4
Gateway 5

The ALB distributes traffic:

Request 1 → Gateway 1
Request 2 → Gateway 3
Request 3 → Gateway 2
Request 4 → Gateway 5

This prevents the gateway itself from becoming a bottleneck.

But what about AWS API Gateway?

Here's an important distinction that often comes up in interviews.

If you're using AWS API Gateway (the managed AWS service):

Internet
    │
    ▼
AWS API Gateway
    │
    ▼
Lambda / ECS / EC2

You generally do not put an ALB in front of AWS API Gateway.

Why?

Because AWS API Gateway is already a fully managed, highly available service. AWS automatically scales it, load balances it, and handles failures for you.

If you're using AWS API Gateway (managed service), then you usually do not need an ALB in front of it.


## Real-world Azure architecture.

Suppose your architecture is:

Internet
      │
      ▼
Azure Front Door
      │
      ▼
Azure API Management (APIM)
      │
      ▼
AKS

The answer is:

Usually, NO. You don't deploy your own Azure Load Balancer separately.

Here's why.

What each component does
1. Azure Front Door

Acts as the global entry point.

Responsibilities:

✅ Global load balancing
✅ WAF
✅ SSL termination
✅ CDN
✅ Health probes
✅ Route users to the nearest healthy region

User

↓

Azure Front Door

↓

Region A
or
Region B
2. Azure API Management (APIM)

Responsibilities:

Authentication
Rate limiting
API keys
Request validation
Caching
API versioning
Routing

It is your API Gateway.

3. AKS

Inside AKS, your application runs in Pods.

Product Pod 1
Product Pod 2
Product Pod 3

How does APIM reach them?

Through a Kubernetes Service (or an Ingress Controller if you're exposing HTTP/HTTPS).

APIM
   │
   ▼
Ingress / Kubernetes Service
   │
 ┌─┴─────────┐
 ▼           ▼
Pod1       Pod2

The Kubernetes Service already load balances requests across the Pods.

So do I need Azure Load Balancer?

Usually no.

Because:

Front Door balances globally.
APIM manages APIs.
Kubernetes Service distributes traffic across Pods.

Adding another manual Azure Load Balancer is often unnecessary.

Complete Flow
               Internet
                    │
                    ▼
          Azure Front Door
                    │
                    ▼
       Azure API Management
                    │
                    ▼
      AKS Ingress / Service
                    │
        ┌───────────┴───────────┐
        ▼                       ▼
    Product Pod 1          Product Pod 2

Notice that Kubernetes Service is already acting as the internal load balancer for the Pods.

Interview Point

If the interviewer asks:

"Where is the load balancing happening?"

You can answer:

Azure Front Door → Global load balancing between regions or backends.
Kubernetes Service / Ingress → Load balancing between Pods inside the AKS cluster.

