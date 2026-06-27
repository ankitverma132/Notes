# **Loadbalancing with ALB & NLB**

Load balancing is the process of distributing incoming requests or workloads across multiple servers or resources so that no single resource becomes overloaded, improving performance, availability, and reliability.

Interview definition (worth remembering)

A load balancer sits between clients and backend servers, distributing incoming requests among healthy servers using algorithms like Round Robin or Least Connections to improve scalability, availability, and fault tolerance.

Example

Suppose your website receives 9,000 requests per second.

Without a load balancer:

          9000 Requests
                │
                ▼
           Server 1 ❌

One server handles everything and may become slow or crash.

With a load balancer:

           9000 Requests
                 │
                 ▼
          Load Balancer
         /      |      \
        ▼       ▼       ▼
   Server 1  Server 2  Server 3
    3000      3000      3000

The requests are distributed, so each server handles only 3,000 requests.


Why do we need a Load Balancer?
✅ Prevents one server from being overloaded.
✅ Improves response time.
✅ Increases availability.
✅ Enables horizontal scaling (adding more servers).
✅ Can stop sending traffic to unhealthy servers (health checks).


## One of the most important jobs of a load balancer is to track the health of backend servers.

How does it work?

The load balancer periodically sends health check requests to each server.

For example:

Every 10 seconds

Load Balancer
     │
     ├── GET /health → Server 1 ✅ 200 OK
     ├── GET /health → Server 2 ✅ 200 OK
     └── GET /health → Server 3 ❌ No Response

If Server 3 doesn't respond or returns an error (like 500 Internal Server Error), the load balancer marks it as unhealthy.

Now, when users send requests:

           Users
             │
             ▼
      Load Balancer
        │       │
        ▼       ▼
   Server 1  Server 2
      ✅         ✅

Server 3 ❌ (No traffic sent)

The load balancer stops sending traffic to Server 3 until it becomes healthy again.

When Server 3 recovers

The health checks continue in the background.

Load Balancer
      │
      └── GET /health → Server 3 ✅ 200 OK

Once Server 3 starts responding successfully again, the load balancer automatically adds it back to the pool.

           Users
             │
             ▼
      Load Balancer
     /      |       \
    ▼       ▼        ▼
Server1  Server2  Server3
  ✅        ✅        ✅
What is a health check endpoint?

Applications usually expose a simple endpoint like:

GET /health

If the application is working:

HTTP/1.1 200 OK

Healthy

If it's not ready or has a problem:

HTTP/1.1 503 Service Unavailable

The load balancer uses this response to decide whether to send traffic to that server.

## Properties of load balancer:
Automatic distribution of incoming traffic accros multiple target
Monitor health of targets
Integrate with SSL
Elastc - Means as traffic goes up, load balancer will also scale up



## Type of loadbalancers:

The main types are based on which OSI layer they operate on.

1. Layer 4 (Transport Layer) Load Balancer(Network load balancer)

Works at the TCP/UDP level. Routes traffic based on protocol and port.

It doesn't inspect the actual HTTP request or its contents. It only looks at information like:

Source IP
Destination IP
Port number

Example:

Client
   │
TCP Connection
   │
   ▼
L4 Load Balancer
   │
 ┌─┴───────┐
 ▼         ▼
Server1  Server2

It simply forwards the TCP connection.

Advantages
✅ Very fast
✅ Low latency
✅ Handles millions of connections
Disadvantages
❌ Cannot route based on URLs or HTTP headers.

Netwrk load balancer can not send request to different backend based on url, because url path is application layer property.
NLB handles spiky traffic very well.
NLB works at transport layer(Layer 4) & supports TCP,UDP,TLS traffic. Ideal for loaw latency and hight throughput apps like gaming and streaming.
NLB allows static IP addrs making them suitable option for apps that needs fixed Ips.

2. Layer 7 (Application Layer) Load Balancer

Works at the HTTP/HTTPS level.

It understands:

URL paths
Headers
Cookies
Hostnames
HTTP methods

Example:

Request:
GET /payments

        │
        ▼
Layer 7 Load Balancer
        │
        └──► Payment Service

Another request:

GET /products

        │
        ▼
Layer 7 Load Balancer
        │
        └──► Product Service

This is called content-based routing.

Example

Suppose you have:

shop.com/products
shop.com/orders
shop.com/payments

The Layer 7 load balancer routes them like this:

                shop.com
                    │
                    ▼
          Layer 7 Load Balancer
         /         |           \
        ▼          ▼            ▼
Product      Order Service   Payment
 Service

The user still sees a single domain (shop.com), but requests go to different backend services.

Layer 4 vs Layer 7
Feature	                Layer 4	        Layer 7
Works on	            TCP/UDP	        HTTP/HTTPS
Speed	                Faster	        Slightly slower
Understands URLs	    ❌ No	        ✅ Yes
Header-based routing	❌ No	        ✅ Yes
Cookie-based routing	❌ No	        ✅ Yes
SSL termination	Usually no	             ✅ Yes
Best for	            Raw TCP traffic	Web applications & APIs

## Popular Load Balancers

Some widely used load balancers include:

Nginx (mostly Layer 7)
AWS Application Load Balancer (ALB) → Layer 7
AWS Network Load Balancer (NLB) → Layer 4
Google Cloud Load Balancer
Azure Load Balancer


ALB handles Http & Https traffic application layer.
Alb can route traffic based on url, query param, host name, http method, sorce IP, or port number.
ALB are best for microservices arch. due to advance routing capability.
Both ALB & NLB handles SSL/TLS encryption.


ALB specialized in handling http & https traffic. NLB specialized in TCP, UDP & Tls traffic.





Which one is used with Kubernetes?

In Kubernetes, you commonly see:

Internet
    │
    ▼
External Load Balancer
    │
    ▼
Ingress Controller (Nginx/Envoy)
    │
 ┌──┼────────────┐
 ▼  ▼            ▼
Product     Order     Payment
Service     Service   Service

The Ingress Controller often acts as a Layer 7 load balancer, routing requests to the correct Kubernetes Service based on paths or hostnames.




## AWS Application Load Balancer (ALB) provides much more than just request distribution.

Here are the important features:

1. Web Application Firewall (WAF) ✅

You can attach AWS WAF to an ALB.

It protects your application from attacks like:

SQL Injection
Cross-Site Scripting (XSS)
Bad bots
IP blocking
Rate limiting

Example:

User
   │
   ▼
AWS WAF
   │
   ▼
ALB
   │
Backend Servers

If someone sends:

' OR 1=1 --

The WAF can block the request before it reaches your application.


2. Authentication ✅

ALB can authenticate users before forwarding requests.

It supports identity providers such as:

OpenID Connect (OIDC)
OAuth 2.0
Amazon Cognito

Flow:

User
   │
   ▼
ALB
   │
Authenticate User
   │
   ▼
Google/Cognito
   │
Success
   │
   ▼
Application

Your backend doesn't always have to implement this login flow itself.

3. Sticky Sessions (Session Affinity) ✅

Normally:

Request 1 → Server A
Request 2 → Server B
Request 3 → Server C

With sticky sessions:

User A

Request 1 → Server A
Request 2 → Server A
Request 3 → Server A

The load balancer uses a cookie so the same client keeps reaching the same backend server for a configurable period.

When is it useful?

Legacy applications that store session data in server memory.
Applications that aren't fully stateless.

Modern recommendation: Prefer storing sessions in a shared store like Redis or using JWTs so any server can handle any request. Then sticky sessions usually aren't necessary.

4. SSL/TLS Termination ✅

Instead of every server decrypting HTTPS:

Client (HTTPS)
      │
      ▼
     ALB
Decrypt HTTPS
      │
HTTP
      │
Backend Servers

The ALB handles the TLS handshake and decryption, reducing work for backend servers.

## What is SSL/TLS Encryption?

Normally, when you visit a website using HTTP:

Browser
   │
   │  Username: ankit
   │  Password: 123456
   ▼
Internet

The data is sent in plain text. Anyone intercepting the traffic could read it.

With HTTPS (TLS/SSL):

Browser
   │
   │  🔒 Encrypted Data
   ▼
Internet

The browser encrypts the data before sending it, and only the server can decrypt it using its TLS certificate and private key.

This protects:

Passwords
Credit card information
Cookies
API tokens
Personal information

What is SSL/TLS Termination?

Suppose you have three backend servers.

Without SSL termination:

Client (HTTPS)
       │
       ▼
   Server 1
Decrypt HTTPS

Client (HTTPS)
       │
       ▼
   Server 2
Decrypt HTTPS

Client (HTTPS)
       │
       ▼
   Server 3
Decrypt HTTPS

Every server performs the expensive TLS handshake and decryption.

With an ALB:

Client
   │ HTTPS
   ▼
ALB
Decrypts HTTPS
   │
HTTP (or HTTPS)
   │
 ┌──────┴──────┐
 ▼             ▼
Server1     Server2

The ALB handles encryption/decryption, so the backend servers can focus on application logic.

Note: If you require end-to-end encryption, the ALB can also forward traffic to the backend over HTTPS instead of HTTP.

Does NLB support SSL/TLS?

Yes. This is a common interview point.

Both ALB and NLB support TLS, but differently.

Feature	                ALB	    NLB
HTTPS support	        ✅	    ✅
TLS termination	        ✅	    ✅
TCP	                    ❌	    ✅
UDP	                    ❌	    ✅
HTTP-aware	            ✅	    ❌
Path-based routing	    ✅	    ❌
Host-based routing	    ✅	    ❌



## The server decrypts the request using its private key, which matches the public key used by the browser for encryption.

Step 1: Browser wants to connect

You type:

https://amazon.com

Your browser says:

"I want to talk securely."

Step 2: Server sends its certificate

The server (or ALB) replies:

Certificate

Public Key 🔑

Issued by:
Let's Encrypt / DigiCert / Amazon

Notice:

The server does NOT send its private key.

It only sends:

Certificate
Public Key

Step 3: Browser verifies the certificate

The browser checks:

Is it expired?
Is it issued by a trusted Certificate Authority (CA)?
Does it belong to amazon.com?

If yes:

✓ Certificate trusted
Step 4: Browser creates a secret

The browser generates a random value, often called a session key.

Example:

ABCD1234XYZ

This key is used for encrypting the rest of the communication.

Step 5: Browser encrypts the secret

The browser encrypts this session key using the server's public key.

Session Key

ABCD1234XYZ

↓

Encrypted using Public Key

98ashd81asd...

It sends the encrypted version to the server.

Step 6: Server decrypts it

Only the server has the matching private key.

Encrypted Secret

98ashd81asd...

↓

Private Key

↓

ABCD1234XYZ

No one else can decrypt it.

Even if a hacker intercepts the encrypted message:

98ashd81asd...

they can't recover the session key without the private key.

Step 7: Secure communication begins

Now both the browser and the server know:

Session Key

ABCD1234XYZ

From this point on, they use this shared session key with symmetric encryption, which is much faster than public-key encryption.

Browser
     │
Encrypt using Session Key
     │
Internet
     │
Decrypt using Session Key
     ▼
Server
Why don't we use the public/private keys for every request?

Because public-key (asymmetric) encryption is computationally expensive.

Instead:

Use asymmetric encryption once during the TLS handshake to securely exchange a session key.
Use symmetric encryption (the session key) for all subsequent data because it's much faster.]