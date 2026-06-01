# **Client Side Data Fetching**

Client-side data fetching in Next.js means:
Browser fetches data AFTER page loads.
Very similar to React.

🚀 Basic Flow
Browser → API → Update UI

🧠 Usually done using
useEffect
fetch
axios

🔥 Basic Example
import { useEffect, useState } from "react"

export default function Users() {
  const [users, setUsers] = useState([])

  useEffect(() => {
    fetch("https://jsonplaceholder.typicode.com/users")
      .then((res) => res.json())
      .then((data) => setUsers(data))
  }, [])

  return (
    <div>
      {users.map((user) => (
        <p key={user.id}>{user.name}</p>
      ))}
    </div>
  )
}

⚡ What happens here?
1. Page renders first

Initially:

users = []
2. Browser calls API

Inside:

useEffect()
3. State updates
setUsers(data)
4. UI re-renders

Users appear.

🚀 Async/Await Version (cleaner)
import { useEffect, useState } from "react"

export default function Users() {
  const [users, setUsers] = useState([])

  useEffect(() => {
    async function fetchUsers() {
      const res = await fetch(
        "https://jsonplaceholder.typicode.com/users"
      )

      const data = await res.json()

      setUsers(data)
    }

    fetchUsers()
  }, [])

  return (
    <div>
      {users.map((user) => (
        <p key={user.id}>{user.name}</p>
      ))}
    </div>
  )
}

🔥 Why called “client-side”?
Because fetching happens:
👉 in browser/client

NOT on server.

⚠️ Drawbacks
Initial page loads without data

Can cause:
loading state
SEO not ideal

🚀 Usually add loading state
const [loading, setLoading] = useState(true)
Example
if (loading) return <p>Loading...</p>

🧠 Difference from Server-side fetching
Client-side
Browser fetches after render
Server-side
Server fetches before sending HTML

💬 When YOU should use client-side fetching

Perfect for:
dashboards
authenticated apps
user-specific data
dynamic UI

## SWR
SWR in Next.js is a React data-fetching library made by Vercel 🔥

It helps with:
API fetching
caching
auto revalidation
loading/error handling

WITHOUT writing lots of useEffect boilerplate.

🚀 Full Form
SWR
Stale While Revalidate

Sounds complex 😄 but simple idea:

Show cached data first
Then fetch fresh data in background

🧠 Why people use SWR

Instead of writing:

useEffect()
useState()
loading state
error state

SWR handles most automatically.

🚀 Basic Example
Install:
npm install swr

Usage
import useSWR from "swr"

const fetcher = (url) => fetch(url).then(res => res.json())

export default function Users() {
  const { data, error, isLoading } = useSWR(
    "/api/users",
    fetcher
  )
  if (isLoading) return <p>Loading...</p>
  if (error) return <p>Error...</p>

  return (
    <div>
      {data.map(user => (
        <p key={user.id}>{user.name}</p>
      ))}
    </div>
  )
}

🔥 What SWR gives automatically
✅ Caching

Data reused automatically.

✅ Re-fetching

When tab refocuses:
👉 data refreshes.

✅ Loading state
isLoading

✅ Error handling
error
✅ Cleaner code

Less boilerplate.

🧠 Traditional Fetch vs SWR
Without SWR
useState
useEffect
loading
error
manual caching

With SWR
useSWR()

⚡ What “Stale While Revalidate” means

Suppose old cached data exists.

SWR:
Shows old data immediately ⚡
Fetches fresh data in background 🔄
Updates UI automatically ✅

Very fast user experience.

Cached data - previously fetched data stored temporarily 🔥

🚀 Example Without Cache

Suppose dashboard loads:

GET /workouts

Every time:

page opens
tab switches
component rerenders

👉 API called again and again.

Slow + unnecessary.

🔥 With Cache

First API call result gets stored.

Example data:

[
  { id: 1, name: "Chest" },
  { id: 2, name: "Legs" }
]

SWR keeps this in memory.

🧠 Next time component loads

Instead of:

wait for API

SWR immediately shows:

cached old data

VERY fast ⚡

Then in background

SWR silently calls API again:

fetch latest data

If new data comes:
👉 UI updates automatically.
