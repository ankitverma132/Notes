# **Authentication**

It handles:
login
signup
sessions
OAuth
JWT
protected routes

without building auth from scratch.

🚀 What NextAuth Provides
✅ Authentication providers

Like:

Google login
GitHub login
Credentials login
Microsoft login
✅ Session management

Keeps user logged in.

✅ JWT handling
✅ Secure cookies
✅ Route protection

## 🧠 Basic Flow
User Login
    ↓
NextAuth validates
    ↓
Session/JWT created
    ↓
User stays authenticated

🚀 Installation
npm install next-auth

## 🚀 Setup API Route
Pages Router:
pages/api/auth/[...nextauth].js

## Basic Example
import NextAuth from "next-auth"
import GitHubProvider from "next-auth/providers/github"

export default NextAuth({
  providers: [
    GitHubProvider({
      clientId: process.env.GITHUB_ID,
      clientSecret: process.env.GITHUB_SECRET
    })
  ]
})

🧠 Why [...nextauth]?
Catch-all route handles:
/api/auth/signin
/api/auth/signout
/api/auth/session

all automatically.

🚀 Wrap App
import { SessionProvider } from "next-auth/react"

export default function App({
  Component,
  pageProps
}) {
  return (
    <SessionProvider session={pageProps.session}>
      <Component {...pageProps} />
    </SessionProvider>
  )
}

## 🚀 Login Button
import { signIn, signOut, useSession }
from "next-auth/react"

export default function Home() {
  const { data: session } = useSession()

  if (session) {
    return (
      <>
        <p>Welcome {session.user.name}</p>

        <button onClick={() => signOut()}>
          Logout
        </button>
      </>
    )
  }

  return (
    <button onClick={() => signIn()}>
      Login
    </button>
  )
}

## 🧠 useSession()
Gives:

logged in user
session info
loading state

🔥 Session vs JWT

## NextAuth supports:

database sessions
JWT sessions

JWT commonly used nowadays.

## 🚀 Protecting Pages

Client-side:

const { data: session } = useSession()

if (!session) {
  return <p>Not authorized</p>
}

SSR Protection
getServerSession()

used in server-side rendering.

SSR protection means:

protecting pages BEFORE page is sent to browser.

Using server-side authentication check.

🚀 Problem Without SSR Protection

Suppose protected page:

/dashboard
Client-side protection
useSession()

Problem:

page may start rendering first
then auth check happens

Sometimes:
👉 small flash of protected UI visible.

🔥 SSR Protection Fixes This

Before sending page:

Server checks authentication

If user NOT logged in:
👉 redirect immediately.

Browser never gets protected page.

🚀 In NextAuth
Use:
getServerSession()

inside:
getServerSideProps()

Example
import { getServerSession }
from "next-auth/next"

export async function getServerSideProps(context) {
  const session = await getServerSession(
    context.req,
    context.res,
    authOptions
  )

  if (!session) {
    return {
      redirect: {
        destination: "/login",
        permanent: false
      }
    }
  }

  return {
    props: {}
  }
}
export default function Dashboard() {
  return <h1>Dashboard</h1>
}

🧠 Flow
Request /dashboard
       ↓
Server checks session
       ↓
Logged in?
   ↙         ↘
 Yes          No
 ↓             ↓
Send page    Redirect login

🔥 Why this is better
Security

Protected HTML never reaches browser.

✅ No UI flicker
✅ Better UX

## ⚡ Important Understanding
NextAuth is mainly:
authentication/session framework

NOT full backend auth server.