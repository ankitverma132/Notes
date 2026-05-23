# **Azure Application Gateway**

## 🧠 What is Application Gateway?

It is:

Layer 7 Load Balancer

for HTTP/HTTPS traffic.

🚀 Main job

It sits in front of your applications and handles:

Internet Traffic
      ↓
Application Gateway
      ↓
Backend Apps
🔥 Why not normal Load Balancer?

Because Azure Load Balancer works at:

Layer 4 (TCP/UDP)

Application Gateway works at:

Layer 7 (HTTP/HTTPS)

So it understands:

URLs
paths
domains
cookies
headers
🧠 Real example

Suppose:

api.health.com/auth
api.health.com/survey
api.health.com/reports

Application Gateway can route:

Path	Backend
/auth	auth-service
/survey	survey-service
/reports	report-service
🚀 Architecture
Users
   ↓
Application Gateway
   ↓
AKS / App Service / VMs
🔥 Major features
1️⃣ Path-based routing

Example:

/api/auth    → auth-service
/api/report  → report-service
2️⃣ SSL termination

Gateway handles HTTPS.

HTTPS
   ↓
Application Gateway decrypts
   ↓
HTTP internally

Backend apps don’t manage certificates directly.

3️⃣ Web Application Firewall (WAF)

VERY important feature 🔥

Protects against:

SQL injection
XSS
malicious traffic
4️⃣ Load balancing

Distributes traffic across instances.

5️⃣ Health probes

Checks:

is backend healthy?

If unhealthy:

stop routing traffic there
🧠 Typical AKS architecture
Internet
   ↓
Application Gateway
   ↓
Ingress Controller
   ↓
AKS Services
   ↓
Pods

## 🚀 Compared to Azure Load Balancer
Azure LB	Application Gateway
Layer 4	Layer 7
TCP/UDP	HTTP/HTTPS
simple balancing	intelligent routing
no URL awareness	path/domain aware


⚡ Architecture progression

Usually:

Public IP
   ↓
Application Gateway
   ↓
AKS / App Services


🧠 One-line takeaway

Azure Application Gateway is a Layer 7 web traffic load balancer that intelligently routes HTTP/HTTPS traffic to backend applications with features like SSL termination, WAF, and path-based routing.



🧠 Simplified flow
Browser / UI
      ↓
Application Gateway
      ↓
AKS Service / Ingress
      ↓
Pod



## 🚀 Real example

Suppose user opens:

https://healthapp.com/survey
🔥 Flow step-by-step
1️⃣ Browser sends request
GET /survey

to:

Application Gateway public IP/domain


2️⃣ Application Gateway receives traffic

It checks rules like:

Path	Route to
/auth	auth-service
/survey	survey-service


3️⃣ Gateway forwards to AKS

Usually through:

Ingress Controller

or directly to:

Kubernetes Service


4️⃣ Kubernetes Service routes traffic

Service finds healthy pod.

Example:

survey-service
    ↓
survey-pod-1


5️⃣ Pod handles request

Spring Boot container processes API.

🧠 Full realistic architecture
User Browser
      ↓
DNS
      ↓
Application Gateway
      ↓
Ingress Controller
      ↓
Kubernetes Service
      ↓
Pod


🚀 Think of responsibilities
Component	Responsibility
Application Gateway	internet traffic routing
Ingress	K8s HTTP routing
Service	stable endpoint for pods
Pod	actual application



🧠 One-line takeaway

The UI/browser sends requests to Application Gateway, which routes traffic into AKS where Kubernetes Services forward requests to healthy pods.

## Load balancing: 

Application Gateway load balances:
traffic entering the cluster/app

Kubernetes Service load balances:
traffic between pods internally

🚀 Think of two stages
🌍 Stage 1 — External Traffic Balancing

Handled by:

Azure Application Gateway

Example:

User requests coming from internet

Gateway decides:

which app
which backend service
which AKS ingress
🌍 Stage 2 — Internal Pod Balancing

Handled by:

Kubernetes Service

Service decides:

which healthy pod gets request


🧠 Example

Suppose:

3 survey-service pods
survey-pod-1
survey-pod-2
survey-pod-3
🚀 What Application Gateway does

It routes:

/survey

to:

survey-service

inside AKS.

🚀 What Kubernetes Service does

Then Kubernetes Service load balances:

survey-service
      ↓
one of healthy survey pods
🔥 Important distinction

Application Gateway usually DOES NOT know:

individual pod IPs

Kubernetes handles pod discovery.

🧠 Why this separation exists

Because pods:

scale dynamically
restart often
IPs change constantly

Kubernetes is much better at handling:

internal pod orchestration

🚀 Application Gateway focuses on
Feature	            Example
SSL termination	    HTTPS
WAF	                security
path routing	    /auth vs /survey
domain routing	    api vs admin
external balancing	multi backend

🚀 Kubernetes Service focuses on
Feature	        Example
pod discovery	dynamic pods
internal LB	healthy pod selection
cluster networking	pod communication


🧠 One-line takeaway

Application Gateway balances and routes incoming internet traffic to AKS services/ingress, while Kubernetes Services load balance requests across healthy pods internally.



🔥 Application Gateway health checks

Application Gateway checks:

Is backend endpoint reachable and healthy?

Example:

Can I reach survey-service?

Scenario 1 — One pod unhealthy
pod-2 crashes

Kubernetes removes it from service endpoints.

Application Gateway may never even notice.

Scenario 2 — Entire service/ingress unhealthy

Example:

ingress controller crashed
node issue
service unreachable

Then:

Application Gateway detects backend unhealthy

and stops routing there.


🔥 Example health endpoint

App Gateway may probe:

/health

or:

/actuator/health


🚀 If probe fails

Application Gateway marks backend:
Unhealthy
and avoids routing traffic.


Application Gateway
"Is the building reachable?"
Kubernetes
"Which employees inside are available?"



Application Gateway performs higher-level backend health checks, while Kubernetes handles detailed pod-level health and traffic routing internally.



## Creating Application gateway:

🧠 Before creating App Gateway

You need:

Requirement	            Why
Virtual Network (VNet)	networking
Dedicated subnet	    App Gateway requires own subnet
Public IP	            internet access
Backend app	               AKS/App Service/VM


🚀 Architecture
Internet
   ↓
Public IP
   ↓
Application Gateway
   ↓
Backend Pool
   ↓
AKS / App Service


🔥 Important rule
Application Gateway MUST have:
its own dedicated subnet

Very important Azure rule.


## 🚀 Step-by-step in Azure Portal
1️⃣ Create Virtual Network

Go to:

Azure Portal
 → Virtual Networks
 → Create

Example:

Setting	Value
Name	health-vnet
Address space	10.0.0.0/16

### Vnet & Subnet:

🧠 Start from real-world analogy

Think of Azure cloud like:

A huge city
🚀 VNet = Your Private Colony/Society

A:
Azure Virtual Network

is your own private network space inside Azure.

🧠 Meaning

Resources inside same VNet can communicate privately.

Example:

AKS
Database
Application Gateway
VMs

can talk securely internally.

🔥 Without VNet

Everything would communicate:

over public internet ❌

which is:

insecure
slower
harder to control
🚀 Example
health-vnet

may have IP range:

10.0.0.0/16

This means:

all internal resources get private IPs from this range
🧠 Then what is subnet?

Subnet = smaller division INSIDE VNet.

🚀 Analogy
VNet = apartment complex
Subnet = individual blocks/floors
🔥 Example
health-vnet
   ├── appgw-subnet
   ├── aks-subnet
   └── db-subnet

Each subnet isolates certain resources.

🧠 Why subnet needed?

Because different services have:

different security rules
traffic patterns
scaling behavior

🚀 Example separation
Subnet	        Resources
appgw-subnet	Application Gateway
aks-subnet	    AKS nodes
db-subnet	    Databases

🔥 Why Application Gateway NEEDS subnet

Because:
Azure Application Gateway

is actually:

a managed networking appliance

internally running Azure-managed instances.

It needs:

private IPs
scaling IPs
routing tables
network isolation

🧠 Azure internally allocates

Application Gateway instances inside subnet.

Example:

appgw-subnet
   ↓
App Gateway instance 1
App Gateway instance 2

🚀 Why dedicated subnet?

Because App Gateway:

scales dynamically
reserves IPs
controls routing internally

Azure wants:

clean isolated networking

🔥 Real architecture
Internet
   ↓
Public IP
   ↓
Application Gateway (appgw-subnet)
   ↓
AKS Cluster (aks-subnet)
   ↓
Pods

🧠 Why VNet overall is important

VNet provides:

private communication
network isolation
security boundaries
routing control

Without VNet:

cloud networking would be chaos
🚀 Important networking hierarchy
Azure Region
    ↓
VNet
    ↓
Subnets
    ↓
Resources

🧠 One-line takeaway

A VNet provides a private network space in Azure, while subnets divide that network into isolated sections for different resources like AKS and Application Gateway.


2️⃣ Create subnets

Example:

Subnet	Purpose
appgw-subnet	Application Gateway
aks-subnet	AKS
🚀 Example
10.0.1.0/24 → appgw-subnet
10.0.2.0/24 → aks-subnet
3️⃣ Create Application Gateway

Go to:

Azure Portal
 → Application Gateway
 → Create
4️⃣ Basics tab

Example:

Setting	Value
Name	health-app-gateway
Tier	Standard V2
Region	East US
🧠 Standard vs WAF
Tier	Purpose
Standard	normal routing
WAF	security protection

For production:

WAF recommended
5️⃣ Frontend IP

Choose:

Public IP

Azure creates internet-facing endpoint.

6️⃣ Backend Pool

This is:

where traffic should go

Can be:

AKS ingress
App Service
VM IPs
7️⃣ Routing rules

Example:

Path	Backend
/api	backend-api
/auth	auth-service
🔥 Example full routing
healthapp.com/api
       ↓
Application Gateway
       ↓
AKS backend service
🚀 For AKS specifically

Usually:

install ingress controller
connect App Gateway using AGIC
🧠 AGIC

AGIC =

Application Gateway Ingress Controller

It automatically syncs:

Kubernetes ingress rules
      ↓
Application Gateway config

Very powerful.

🚀 Azure CLI example

Basic creation:

az network application-gateway create \
  --name health-app-gateway \
  --location eastus \
  --resource-group acr-learning-rg \
  --vnet-name health-vnet \
  --subnet appgw-subnet \
  --capacity 2 \
  --sku Standard_v2 \
  --public-ip-address health-appgw-ip
🔥 Important real-world architecture

Usually:

Internet
   ↓
Azure DNS
   ↓
Application Gateway
   ↓
AKS Ingress
   ↓
Services
   ↓
Pods
🧠 What App Gateway stores internally
frontend listener
backend pool
health probes
routing rules
SSL certs
🚀 Big picture

Application Gateway acts like:

smart HTTP traffic manager

between internet and your apps.

🧠 One-line takeaway

To create an Azure Application Gateway, you first need a VNet and dedicated subnet, then configure frontend IPs, backend pools, and routing rules to direct HTTP/HTTPS traffic to your applications. 


                         ┌──────────────────────┐
                         │      End Users       │
                         │  Browser / Mobile    │
                         └──────────┬───────────┘
                                    │
                                    │ HTTPS Request
                                    ▼
                    ┌────────────────────────────────┐
                    │      Azure Application         │
                    │          Gateway               │
                    │--------------------------------│
                    │ • SSL Termination              │
                    │ • Path Routing                 │
                    │ • WAF / Security               │
                    │ • Health Checks                │
                    └──────────┬─────────────────────┘
                               │
          ┌────────────────────┴────────────────────┐
          │                                         │
          │                                         │
          ▼                                         ▼

┌───────────────────────┐              ┌───────────────────────────┐
│ Azure App Service     │              │ Azure Kubernetes Service  │
│-----------------------│              │            (AKS)          │
│ UI React App          │              │---------------------------│
│ Running as Docker     │              │ Backend Microservices     │
│ Container             │              │ Running as Pods           │
└──────────┬────────────┘              └────────────┬──────────────┘
           │                                        │
           │ Pull Docker Image                      │ Pull Docker Images
           ▼                                        ▼

      ┌────────────────────────────────────────────────────┐
      │        Azure Container Registry (ACR)             │
      │---------------------------------------------------│
      │ Stores Docker Images                              │
      │                                                   │
      │ • react-ui:v1                                     │
      │ • auth-service:v1                                 │
      │ • survey-service:v1                               │
      │ • report-service:v1                               │
      └────────────────────────────────────────────────────┘


                         ┌─────────────────────────┐
                         │     Application         │
                         │       Insights          │
                         │-------------------------│
                         │ Logging                 │
                         │ Metrics                 │
                         │ Distributed Tracing     │
                         │ Exceptions              │
                         │ Performance Monitoring  │
                         └──────────┬──────────────┘
                                    ▲
            ┌───────────────────────┼────────────────────────┐
            │                       │                        │
            │                       │                        │
            │ Telemetry             │ Telemetry              │ Telemetry
            │                       │                        │
            ▼                       ▼                        ▼

   ┌────────────────┐    ┌────────────────┐      ┌────────────────┐
   │ React UI       │    │ AKS Services   │      │ Application    │
   │ App Service    │    │ Pods           │      │ Gateway Logs   │
   └────────────────┘    └────────────────┘      └────────────────┘



   ![alt text](image-22.png)

   🚀 Real enterprise flow
User
 ↓
Application Gateway (public entry)
 ↓
UI App Service
 ↓
Private VNet traffic
 ↓
AKS Internal LoadBalancer/Ingress
 ↓
Pods


🧠 Final simplified architecture to remember
ACR → stores images

App Gateway → entry point

App Service → UI app

AKS → backend services

Application Insights → monitoring


🚀 Your architecture in simple words
1️⃣ ACR
Stores Docker images

Like:

react-ui:v1
auth-service:v1
2️⃣ App Service
Runs your React UI container

Easy hosting.

3️⃣ AKS
Runs backend containers/pods

For microservices.

4️⃣ Application Gateway
Single entry point for internet traffic

Routes requests correctly.

5️⃣ Application Insights
Monitoring + logs + tracing
🧠 Actual request flow
User opens website
      ↓
Application Gateway
      ↓
React UI (App Service)
      ↓
React calls backend APIs
      ↓
AKS backend services


## WAF Rules:
WAF rules are security rules inside:

Azure Web Application Firewall

that protect your applications from common web attacks.

Usually WAF is enabled on:
Azure Application Gateway
Front Door
CDN

Application Gateway = security guard
WAF rules = suspicious behavior detection rules



Application Gateway vs frontdor:

🧠 Simplest difference
Service	Scope
Azure Application Gateway	Regional
Azure Front Door	Global
🚀 Mental model
Application Gateway
Traffic manager INSIDE one Azure region

Example:

East US only
Front Door
Global traffic entry point across regions

Example:

India users → India region
US users → US region
🧠 Application Gateway role

Mainly:

Layer 7 routing
WAF
SSL termination
path-based routing

inside one region/VNet.

🧠 Front Door role

Mainly:

global routing
CDN acceleration
edge networking
multi-region failover
🚀 Architecture comparison
🟢 Application Gateway
Users
  ↓
Application Gateway
  ↓
AKS/App Service

Regional setup.

🔵 Front Door
Global Users
      ↓
Azure Front Door
      ↓
East US OR India OR Europe
      ↓
Regional App Gateways / Apps

Global intelligent routing.

🔥 Key difference
Application Gateway

Works:

inside Azure region

Usually inside:

VNet
private networking
Front Door

Works:

at Azure edge locations globally

Closer to users worldwide.

🧠 Example

Suppose your app exists in:

US
Europe
India
Without Front Door

Indian users may accidentally hit:

US backend

High latency.

With Front Door

Front Door routes:

Indian users → India region

Much faster.

🔥 Real enterprise architecture

Very common:

Global Users
      ↓
Azure Front Door
      ↓
Regional Application Gateway
      ↓
AKS/App Services


Azure Front Door handles global user traffic routing and acceleration across regions, while Application Gateway manages regional HTTP/HTTPS traffic routing and security inside a specific Azure environment.


🔥 "Blocked at Front Door"

Meaning:

global edge layer rejected request

Possible reasons:

WAF rule
bot protection
geo restriction
malformed request