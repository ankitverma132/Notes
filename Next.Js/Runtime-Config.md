# **PublicRuntimeConfig and ServerRuntimeCofig**

These are OLD Next.js configuration concepts mostly used in Pages Router era 😄

publicRuntimeConfig
serverRuntimeConfig

came from:
next.config.js
🚀 Why they existed?

To separate:
client-accessible config
server-only config

Example
module.exports = {
  serverRuntimeConfig: {
    secret: "my-secret"
  },

  publicRuntimeConfig: {
    apiUrl: "https://api.com"
  }
}
🔥 Difference
Config	Accessible Where
publicRuntimeConfig	client + server
serverRuntimeConfig	server only

🚀 publicRuntimeConfig
Can be used in browser.

Example:
public API URL
app name
feature flags
Usage
import getConfig from "next/config"

const { publicRuntimeConfig } =
  getConfig()

console.log(publicRuntimeConfig.apiUrl)
⚠️ IMPORTANT

Anything here:
can reach browser bundle
So NEVER put secrets.

🚀 serverRuntimeConfig
ONLY available on server.
Good for:

secrets
tokens
DB credentials
Example
serverRuntimeConfig: {
  dbPassword: "secret"
}

⚠️ But Here’s Important Modern Reality

These configs are now mostly:
deprecated/less preferred

especially in App Router world.

🚀 Modern Next.js Uses
Environment variables instead.

Example
NEXT_PUBLIC_API_URL=https://api.com

JWT_SECRET=abc123
🧠 Rule
NEXT_PUBLIC_*

Accessible in:
client
browser
Normal env vars

Server only.

Example:
DB_PASSWORD=secret
cannot be accessed in browser.

🚀 Browser CANNOT access it
This WON’T work in client component:

"use client"
console.log(process.env.JWT_SECRET)

It becomes:
undefined

in browser bundle.

🔥 Why?
Because Next.js protects non-public env vars.

They are NOT injected into browser JavaScript.

🧠 Simple Rule
Variable	        Browser Access?
JWT_SECRET	        ❌ No
DB_PASSWORD	        ❌ No
NEXT_PUBLIC_API_URL	✅ Yes

🔥 Internal Build Behavior

During build:
Next.js injects NEXT_PUBLIC_*
into client bundle.
But:
normal env vars stay on server

🚀 Modern Usage
process.env.NEXT_PUBLIC_API_URL
or
process.env.JWT_SECRET

🔥 Why env vars replaced runtimeConfig
Simpler.
Works better with:
Vercel
Docker
Kubernetes
Azure
CI/CD

