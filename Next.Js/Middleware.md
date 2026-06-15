# **Middleware**

Middleware in Next.js is basically:
code that runs BEFORE request reaches page/API.

Think of it like:
checkpoint/interceptor
for incoming requests 🔥

🚀 Flow Without Middleware
Request
   ↓
Page/API
   ↓
Response

🚀 Flow With Middleware
Request
   ↓
Middleware
   ↓
Page/API
   ↓
Response

Middleware gets chance to:
inspect request
modify request
redirect
block access

BEFORE actual route executes.

## 🚀 File Location

Usually:
middleware.js

in project root.

Example
import { NextResponse }
from "next/server"

export function middleware(request) {
  console.log("Middleware running")

  return NextResponse.next()
}

🧠 NextResponse.next()
Means:
continue request normally

🔥 SUPER Common Use Case → Authentication

Example:
/dashboard
should only work if logged in.

Example Middleware

import { NextResponse }
from "next/server"

export function middleware(request) {
  const token =
    request.cookies.get("token")

  if (!token) {
    return NextResponse.redirect(
      new URL("/login", request.url)
    )
  }

  return NextResponse.next()
}

🚀 What Middleware Can Access
request headers
cookies
URL
pathname
query params

Example
request.nextUrl.pathname


🚀 Match Specific Routes
export const config = {
  matcher: ["/dashboard/:path*"]
}
Middleware runs ONLY for:

/dashboard/*

🔥 Important Real-world Uses
Use Case	    Example
Auth protection	dashboard/admin
Redirects	    old URL → new URL
Localization	/en, /fr
Logging	        analytics
A/B testing	    feature flags


## Middleware runs BEFORE route handling regardless of whether route is:

SSR
SSG
API route
App Router page
Pages Router page

IF route matches middleware config.

🧠 Even Static Pages?
YES.
Example:
/about

might be fully static generated page.

Still:
request hits middleware first
before serving static file.

🚀 Client-side navigation ALSO triggers it
Example:
router.push("/dashboard")
or:
<Link href="/dashboard" />

Still:
middleware executes

because navigation makes server request internally.

## 🧠 What Middleware Sees

It intercepts:
HTTP request layer
not React rendering layer.

That’s why:
SSR
SSG
App Router
Pages Router

all can pass through it.

🚀 Exception
Middleware only runs:
for matched routes
when request actually happens

Not for:
pure client-side state changes
local React rendering