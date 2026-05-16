# **Azure Key vault**

🧠 What is Azure Key Vault?

👉 It’s a secure service to store:
secrets
passwords
API keys
certificates
connection strings

🔥 Real problem it solves

Without Key Vault:

Backend code
   ↓
Hardcoded secrets ❌
Example:

const storageKey = "abcd123...";
👉 Dangerous:

leaks in GitHub
visible to developers
hard to rotate

🧩 Example secrets
Store:

DB_PASSWORD
STORAGE_ACCOUNT_KEY
JWT_SECRET
API_KEYS

🧠 Real-world architecture

App in AKS
   ↓
Managed Identity
   ↓
Azure Key Vault
   ↓
Gets secrets securely

## 🚀 What you create in Key Vault

Inside Azure Key Vault:

Type	Purpose
Secrets	passwords, tokens
Keys	encryption keys
Certificates	SSL certs
👉 You’ll mostly use:

Secrets

🔐 Example flow (your Blob case)

Instead of:
accountKey in .env
You do:

Key Vault → STORAGE_ACCOUNT_KEY
Backend fetches it securely

## 🚀 How apps access Key Vault

Two common ways:

1. Client Secret / Credentials

App authenticates using:

tenant id
client id
client secret

Managed Identity removes the need to store secrets for accessing Key Vault by letting Azure authenticate the app automatically.

🧠 The problem

If your app needs:

client-secret
to access Key Vault…
then:
Where do we store THAT secret? 🤯
Hardcoding it defeats the purpose.

## ✅ Modern solution: Managed Identity

In Microsoft Azure:

Azure automatically gives your app an identity
No passwords needed.

🧩 Real flow

AKS Pod / App Service / VM
      ↓
Managed Identity
      ↓
Azure AD authenticates automatically
      ↓
Key Vault access granted

🔥 What changes?

Instead of:

clientId + clientSecret
You do:

DefaultAzureCredential()
And Azure handles auth internally.

🚀 Example (Node.js)

import { DefaultAzureCredential } from "@azure/identity";
import { SecretClient } from "@azure/keyvault-secrets";

const credential = new DefaultAzureCredential();

const client = new SecretClient(
  "https://my-keyvault.vault.azure.net/",
  credential
);

const secret = await client.getSecret("STORAGE-KEY");
console.log(secret.value);

🧠 Why this is powerful

No secrets stored anywhere ✅
No rotation headaches ✅
More secure ✅

⚡ Then how does Azure trust the app?

Because:

AKS/App Service/VM has identity enabled
Key Vault grants permissions to that identity

## 🔐 Authorization step
In Key Vault:

Access Control (IAM)
   ↓
Grant:
"Key Vault Secrets User"
to Managed Identity

🧠 Real-world architecture

AKS App
   ↓
Managed Identity
   ↓
Key Vault
   ↓
Get Blob key / DB password

## 🧠 What is Managed Identity?

Managed Identity is an automatically managed Azure identity for your application.

Azure creates an identity for:
AKS
VM
App Service
Function App

so the app can securely access Azure services.


🚀 Example architecture

Your future health app:

Frontend
   ↓
Backend in AKS
   ↓
Managed Identity
   ↓
Key Vault
   ↓
Blob Storage key / DB password

🔥 How authentication works
Your app code:
const credential = new DefaultAzureCredential();

Azure SDK automatically:
detects Managed Identity
gets token from Azure AD
authenticates


🚀 Real code example (Node.js)
Access Key Vault:

import { DefaultAzureCredential } from "@azure/identity";
import { SecretClient } from "@azure/keyvault-secrets";

const credential = new DefaultAzureCredential();

const client = new SecretClient(
  "https://health-app-kv.vault.azure.net/",
  credential
);

const secret = await client.getSecret("storage-account-key");

console.log(secret.value);

🧠 What happens with single AKS identity

Example:

AKS Cluster
   ↓
One Managed Identity

Inside cluster:

Namespace A
  └── Spring Boot App

Namespace B
  └── React App

Namespace C
  └── Worker Service

If all pods use same cluster identity:
All apps inherit same Azure permissions

🔥 Problem
Suppose:
Spring Boot app needs DB password

But:
React app should NOT access DB password
With shared identity:
React app could potentially access it too ❌

🧠 Namespace ≠ Azure Security Boundary

Very important concept:
Kubernetes namespace does NOT isolate Azure identity automatically

Namespaces help organize:
workloads
configs
RBAC داخل K8s

But Azure sees:
same AKS identity
unless configured otherwise.

✅ Modern production solution
Use:
Workload Identity
(or older Pod Identity)

🚀 Better architecture

AKS
 ├── Spring Boot Pod
 │      ↓
 │   Identity A
 │      ↓
 │   Key Vault access to DB secrets
 │
 ├── Worker Pod
 │      ↓
 │   Identity B
 │      ↓
 │   Storage access only
 │
 └── React Pod
        ↓
      No Key Vault access


🧠 This gives:

Benefit	Why important
Least privilege	app gets only needed access
Better security	blast radius reduced
Easier auditing	know who accessed what

If all apps in AKS share one Managed Identity, they share the same Azure permissions; production systems usually use separate workload identities per application for security.


🧠 Use AKS Workload Identity

This lets:
specific Kubernetes pods
      ↓
use specific Azure Managed Identities


instead of sharing one cluster identity.



## What is difference between cluster identity an Workload identity:

The difference is mainly:

Who is accessing Azure resources?
1. Cluster Identity 🔥

Cluster identity belongs to:

AKS Cluster itself

Used by AKS infrastructure/components.

What Cluster Identity Does

Examples:

create load balancer
manage disks
attach networking
interact with Azure resources
sometimes pull images

Think:

AKS platform operations
Architecture
AKS Cluster
   ↓
Cluster Managed Identity
   ↓
Azure APIs
Example

When AKS creates:

Azure Load Balancer
Public IP
Managed Disk

it uses cluster identity.

2. Workload Identity 🔥🔥🔥

Workload identity belongs to:

Specific application/pod

NOT the whole cluster.

What Workload Identity Does

Allows pod/application to access:

Azure Key Vault
Storage
Service Bus
Cosmos DB

securely.

Architecture
Pod
 ↓
Workload Identity
 ↓
Azure AD Token
 ↓
Azure Resource

HUGE Difference

Cluster Identity
Cluster-wide infrastructure identity

Workload Identity
Application-level identity

Fine-grained and secure.

Why Workload Identity Matters

Without it:

Every pod might share same AKS identity

Bad because:

too much access
security risk
violates least privilege
With Workload Identity

Example:

Frontend Pod
   → no Key Vault access

Backend Pod
   → DB secret access

Worker Pod
   → Storage access

Each workload gets separate permissions.

This is enterprise-grade security.

Real Modern AKS Flow
Kubernetes Service Account
   ↓
Federated with Azure AD
   ↓
Mapped to Managed Identity
   ↓
Pod gets Azure token

Very cloud-native architecture.


### 🚀 High-level flow
Spring Boot Pod
    ↓
Kubernetes Service Account
    ↓
Mapped to Azure Managed Identity
    ↓
Gets Azure token
    ↓
Access Key Vault


🧩 Important idea
You do NOT attach identity directly to pod.

You attach:
K8s Service Account
to
Azure Managed Identity

🔥 Architecture example

AKS
 ├── Namespace: backend
 │     └── springboot-app
 │             ↓
 │       ServiceAccount: spring-sa
 │             ↓
 │       Managed Identity: backend-mi
 │
 └── Namespace: frontend
       └── react-app
               ↓
         no Azure identity

2️⃣ Grant permissions

Example:

backend-mi
   ↓
Key Vault Secrets User

3️⃣ Create Kubernetes Service Account

Example:
apiVersion: v1
kind: ServiceAccount
metadata:
  name: spring-sa
  namespace: backend
  annotations:
    azure.workload.identity/client-id: <MANAGED_IDENTITY_CLIENT_ID>


4️⃣ Use Service Account in deployment

spec:
  serviceAccountName: spring-sa


5️⃣ App code uses DefaultAzureCredential()
const credential = new DefaultAzureCredential();

Azure SDK automatically:
detects workload identity
authenticates as that managed identity

🔥 Result
Only THAT app can access:
allowed Azure resources
If your architecture is:

Namespace: stage
   ├── auth-service
   ├── survey-service
   ├── report-service
   └── notification-service


then using:
one Managed Identity for all backend services in that environment
is completely reasonable in many systems.

🧠 Typical environment-based setup

Example

AKS
 ├── namespace: dev
 │      └── shared backend identity
 │
 ├── namespace: stage
 │      └── shared backend identity
 │
 └── namespace: prod
        └── stricter identity

This is where containerization, AKS, identity, and deployment all connect together.

Let’s walk through the FULL flow step-by-step.


🧠 Your setup

You have:

React App
   ↓
Dockerized
   ↓
Image stored in ACR

Now you deploy it to:
Azure Kubernetes Service:
🚀 Step 1 — Build Docker image
React source code
   ↓
Docker build
   ↓
Container image

Example:
health-frontend:v1

🚀 Step 2 — Push to ACR
Azure Container Registry stores the image.

ACR
 └── health-frontend:v1

🚀 Step 3 — AKS deployment YAML
In Kubernetes deployment:

containers:
  - name: frontend
    image: myacr.azurecr.io/health-frontend:v1

AKS now knows:

which image to run

🧠 Important question now

How does AKS pull image from private ACR?

🔥 AKS ↔ ACR connection

You attach ACR to AKS:
az aks update \
  --attach-acr myacr


🧠 What happens internally

AKS Managed Identity
      ↓
Gets permission on ACR
      ↓
Can pull Docker images


🚀 Flow now
React App Source
   ↓
Docker Image
   ↓
ACR
   ↓
AKS pulls image
   ↓
Pod starts

🔥 Now comes Managed Identity question
You asked:
How does app connect to Managed Identity?
🧠 Very important answer
Container image itself:

has NO Azure identity
Identity comes AFTER deployment into AKS.

🚀 Relationship chain
Docker image
   ↓
Pod in AKS
   ↓
Kubernetes Service Account
   ↓
Azure Managed Identity

🧩 In deployment.yaml
Example:
spec:
  serviceAccountName: frontend-sa

🧩 ServiceAccount
apiVersion: v1
kind: ServiceAccount
metadata:
  name: frontend-sa
  annotations:
    azure.workload.identity/client-id: <MANAGED_IDENTITY_CLIENT_ID>
This is where containerization, AKS, identity, and deployment all connect together.

Let’s walk through the FULL flow step-by-step.
🧠 Your setup
You have:
React App
   ↓
Dockerized
   ↓
Image stored in ACR

Now you deploy it to:
Azure Kubernetes Service

🚀 Step 1 — Build Docker image
React source code
   ↓
Docker build
   ↓
Container image

Example:
health-frontend:v1

🚀 Step 2 — Push to ACR
Azure Container Registry stores the image.

ACR
 └── health-frontend:v1

🚀 Step 3 — AKS deployment YAML

In Kubernetes deployment:
containers:
  - name: frontend
    image: myacr.azurecr.io/health-frontend:v1

AKS now knows:

which image to run
🧠 Important question now

How does AKS pull image from private ACR?

🔥 AKS ↔ ACR connection
You attach ACR to AKS:

az aks update \
  --attach-acr myacr

🧠 What happens internally

AKS Managed Identity
      ↓
Gets permission on ACR
      ↓
Can pull Docker images

🚀 Flow now



React App Source
   ↓
Docker Image
   ↓
ACR
   ↓
AKS pulls image
   ↓
Pod starts

🔥 Now comes Managed Identity question

You asked:
How does app connect to Managed Identity?

🧠 Very important answer

Container image itself:
has NO Azure identity
Identity comes AFTER deployment into AKS.

🚀 Relationship chain
Docker image
   ↓
Pod in AKS
   ↓
Kubernetes Service Account
   ↓
Azure Managed Identity

🧩 In deployment.yaml
Example:

spec:
  serviceAccountName: frontend-sa

🧩 ServiceAccount
apiVersion: v1
kind: ServiceAccount
metadata:
  name: frontend-sa
  annotations:
    azure.workload.identity/client-id: <MANAGED_IDENTITY_CLIENT_ID>


This is where containerization, AKS, identity, and deployment all connect together.

🧠 Then inside app code
App simply does:
new DefaultAzureCredential()
Azure SDK:

detects workload identity
fetches token automatically
authenticates to Key Vault


Your React app pod gets linked to a Kubernetes Service Account through the Deployment YAML.


🧠 Main idea

Pod
  ↓
uses ServiceAccount

And that ServiceAccount may be linked to:
Azure Managed Identity

🚀 Step-by-step flow
1️⃣ Create ServiceAccount

Example:

apiVersion: v1
kind: ServiceAccount
metadata:
  name: frontend-sa
  namespace: health-app-dev
  annotations:
    azure.workload.identity/client-id: <MANAGED_IDENTITY_CLIENT_ID>

This says:

frontend-sa
   ↓
represents Azure identity


2️⃣ Reference it in Deployment
Your React deployment:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: react-frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: react-frontend

  template:
    metadata:
      labels:
        app: react-frontend

    spec:
      serviceAccountName: frontend-sa   # 🔥 HERE

      containers:
        - name: react-frontend
          image: myacr.azurecr.io/react-frontend:v1

🔥 THIS line is the connection
serviceAccountName: frontend-sa

It tells Kubernetes:

"Run this pod using frontend-sa identity"

🧠 Behind scenes

When pod starts:

Pod
  ↓
gets ServiceAccount token
  ↓
Azure Workload Identity validates it
  ↓
Maps to Managed Identity


🚀 Then app code works automatically

Inside app:
new DefaultAzureCredential()

Azure SDK detects:
pod identity
and authenticates.

🧩 Full relationship


Deployment
   ↓
serviceAccountName
   ↓
Kubernetes ServiceAccount
   ↓
Azure Managed Identity
   ↓
Azure resources


⚡ Important Kubernetes concept

If you do NOT specify:

serviceAccountName:

then pod uses:
default service account
which usually has no Azure identity mapping.

🧠 Important concept
Creating Managed Identity:
creates identity only

It does NOT automatically access:
Key Vault
Blob Storage
DB
anything

🚀 Where permissions are given?
Permissions are assigned ON the resource.

Example:

Key Vault
   ↓
IAM / RBAC
   ↓
Grant role to Managed Identity


🧩 Real flow

Managed Identity
     ↓
assigned role on
     ↓
Azure Resource


🔥 Example with Key Vault

Suppose you created:
backend-mi

Now you grant:
Key Vault Secrets User
role ON:
health-app-kv

🚀 CLI example

az role assignment create \
  --role "Key Vault Secrets User" \
  --assignee <MANAGED_IDENTITY_PRINCIPAL_ID> \
  --scope $(az keyvault show --name health-app-kv --query id -o tsv)
🧠 Meaning

This says:
backend-mi
   can access
health-app-kv secrets

🔥 Same concept for Blob Storage

Grant role:
Storage Blob Data Contributor
ON:
Storage Account
🚀 Example

az role assignment create \
  --role "Storage Blob Data Contributor" \
  --assignee <MANAGED_IDENTITY_PRINCIPAL_ID> \
  --scope $(az storage account show \
      --name healthappstorageaver132 \
      --resource-group acr-learning-rg \
      --query id -o tsv)


🧠 Architecture mindset
Managed Identity = WHO
RBAC Role = WHAT CAN IT DO
Azure Resource = WHERE

🔥 Visual model

Managed Identity
      │
      │ gets role
      ▼
RBAC Permission
      │
      ▼
Azure Resource

In Kubernetes ServiceAccount YAML, the Azure Managed Identity is linked using an annotation.

🧠 The important line

annotations:
  azure.workload.identity/client-id: <MANAGED_IDENTITY_CLIENT_ID>

This tells Azure:
"This ServiceAccount should use this Azure Managed Identity"

🚀 Full example

Suppose:
Managed Identity name:
backend-mi
Its client ID:

12345678-aaaa-bbbb-cccc-123456789abc

🧩 ServiceAccount YAML

apiVersion: v1
kind: ServiceAccount

metadata:
  name: backend-sa
  namespace: health-app-dev

  annotations:
    azure.workload.identity/client-id: 12345678-aaaa-bbbb-cccc-123456789abc

🔥 This is the actual mapping

backend-sa
    ↓
backend-mi

🚀 Then deployment uses ServiceAccount
apiVersion: apps/v1
kind: Deployment

metadata:
  name: auth-service

spec:
  template:
    spec:
      serviceAccountName: backend-sa

      containers:
        - name: auth-service
          image: myacr.azurecr.io/auth-service:v1


🧠 Full chain now

Pod
 ↓
ServiceAccount (backend-sa)
 ↓
Managed Identity (backend-mi)
 ↓
Azure token
 ↓
Key Vault / Blob / DB

🔍 Where to get client ID?

Run:
az identity show \
  --name backend-mi \
  --resource-group acr-learning-rg \
  --query clientId -o tsv


🧠 Forget Azure for a second
Think only like this:
Pod wants to access Key Vault
Azure asks:

"Who are you?"
Pod itself has no identity.

So Kubernetes says:
"This pod uses ServiceAccount backend-sa"

Then Azure says:
"Oh backend-sa maps to backend-mi identity"

Then Azure checks RBAC:
"backend-mi can access Key Vault"

Done.

🔥 Simplified diagram


Pod
 ↓
ServiceAccount
 ↓
Managed Identity
 ↓
Azure Resource

That’s the entire architecture.


🧩 Think of ServiceAccount like:
Kubernetes username
And Managed Identity like:

Azure username
The annotation connects both.


🚀 Extremely simplified real example

Step 1 — Create Azure identity
backend-mi

Step 2 — Give permission
backend-mi → can access Key Vault

Step 3 — Create K8s ServiceAccount
backend-sa

Connect it to:
backend-mi

Step 4 — Pod uses ServiceAccount
serviceAccountName: backend-sa

🧠 Final meaning
This pod acts as backend-mi identity
That’s literally it.


## A Service Principal is an Azure application identity that authenticates using a client ID, client secret, and tenant ID to securely access Azure resources.

🧠 What is a Service Principal?
Think of it as:
non-human Azure user/account for applications

Instead of:
Ankit logs into Azure

you create:
health-app-sp

for apps/scripts.

🔥 It has 3 important things
Item	Meaning
clientId	username/app ID
clientSecret	password
tenantId	Azure directory


🚀 Authentication flow
App
 ↓
clientId + clientSecret
 ↓
Azure AD
 ↓
Gets access token
 ↓
Calls Key Vault / Blob


🧩 Step 1 — Create Service Principal
Run:
az ad sp create-for-rbac --name health-app-sp

🔥 Output will look like

{
  "appId": "xxxx",
  "displayName": "health-app-sp",
  "password": "xxxx",
  "tenant": "xxxx"
}

🧠 Important mapping

Azure Output	Common Name
appId	clientId
password	clientSecret
tenant	tenantId


🚀 Step 2 — Grant permissions
Right now SP exists but cannot access anything.

Example:

Give Key Vault access:

az role assignment create \
  --role "Key Vault Secrets User" \
  --assignee <APP_ID> \
  --scope $(az keyvault show --name health-app-kv --query id -o tsv)
When you create a Service Principal in Microsoft Azure, Azure generates credentials for the application.


🧠 Creation flow
You run:
az ad sp create-for-rbac --name health-app-sp

🔥 Azure creates
Value	Meaning
appId	client ID
password	client secret
tenant	tenant ID

🧩 Example output
{
  "appId": "1111-2222-3333",
  "password": "abcdxyz",
  "tenant": "9999-8888-7777"
}

🧠 Meaning
Output	Used as
appId	CLIENT_ID
password	CLIENT_SECRET
tenant	TENANT_ID

🚀 Then app uses these
clientId
clientSecret
tenantId

to authenticate with Azure AD.

🔥 Authentication flow
Spring Boot App
      ↓
CLIENT_ID + CLIENT_SECRET
      ↓
Azure AD
      ↓
Access Token
      ↓
Key Vault / Blob Storage
You must grant RBAC permissions.

Example:
health-app-sp
   ↓
Key Vault Secrets User
   ↓
health-app-kv

🔥 Example env vars
AZURE_CLIENT_ID=xxxx
AZURE_CLIENT_SECRET=xxxx
AZURE_TENANT_ID=xxxx

Your Service Principal can now:
prove WHO it is
But Azure still needs to know:
"What is it allowed to access?"
That connection happens through RBAC role assignment.


🧠 Step-by-step relationship

1️⃣ Create Service Principal

health-app-sp
Now Azure knows:
this application identity exists

2️⃣ Service Principal authenticates
Using:

clientId
clientSecret
tenantId

Azure AD gives:
access token

3️⃣ But still NO resource access
By default:
SP cannot access Key Vault

4️⃣ Grant RBAC role on Key Vault
This is the correlation step 🔥

Run:
az role assignment create \
  --role "Key Vault Secrets User" \
  --assignee <CLIENT_ID> \
  --scope $(az keyvault show --name health-app-kv --query id -o tsv)

🧠 Meaning of this command

health-app-sp
   ↓
can access
   ↓
health-app-kv

🔥 Visual relationship


Service Principal
     │
     │ authenticates using secret
     ▼
Azure AD
     │
     │ gets token
     ▼
RBAC Role Assignment
     │
     ▼
Key Vault Access

🚀 Real app flow now
Spring Boot App
      ↓
Uses SP credentials
      ↓
Azure AD validates
      ↓
Gets access token
      ↓
Calls Key Vault
      ↓
RBAC checks permissions
      ↓
Secret returned


🧩 Think of it like office building
Service Principal
Employee ID card

Azure AD
Security gate

RBAC Role
Access permissions

Key Vault
Secure room

### ⚡ Important architecture insight

Authentication:
Who are you?
Authorization:
What can you access?

Azure separates both very clearly.

🔥 Real-world example
One SP may have access to:
Key Vault
Blob Storage
ACR
through separate RBAC assignments.

🧠 One-line takeaway

A Service Principal connects to Key Vault through Azure RBAC role assignments that grant the authenticated identity permission to access the vault.

Your app needs:
CLIENT_ID
CLIENT_SECRET
TENANT_ID

So now the question becomes:
"Where do we store those securely?"

🧠 Common approaches

Method	Used in
.env file	local development
Kubernetes Secret	AKS
CI/CD secret variables	pipelines
Key Vault bootstrap	advanced setups

🧠 Now SP can access Key Vault

🚀 Step 3 — Use in application
Example Node.js:
Install:
npm install @azure/identity @azure/keyvault-secrets

🧩 Code
import { ClientSecretCredential } from "@azure/identity";
import { SecretClient } from "@azure/keyvault-secrets";

const credential = new ClientSecretCredential(
  process.env.TENANT_ID,
  process.env.CLIENT_ID,
  process.env.CLIENT_SECRET
);

const client = new SecretClient(
  "https://health-app-kv.vault.azure.net/",
  credential
);

const secret = await client.getSecret("storage-account-key");

console.log(secret.value);

🔥 Environment variables
Usually stored in:

.env
Kubernetes Secret
CI/CD variables

Example:
TENANT_ID=xxxx
CLIENT_ID=xxxx
CLIENT_SECRET=xxxx

⚠️ Main drawback
You must protect:
CLIENT_SECRET

If leaked:
attacker can access Azure resources

🚀 Real-world usage
Very common in:
GitHub Actions
Jenkins
Azure DevOps
local development
external systems

In our Specialty App it stored under:
Azure Portal under App Service -> Configuration -> Application Settings