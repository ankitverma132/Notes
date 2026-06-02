# **Server-side data fetching**

Server-side data fetching in Next.js means:
Data is fetched on SERVER before page reaches browser

In react you can not do server side renderning.

🚀 Flow
Browser request
      ↓
Next.js server fetches data
      ↓
HTML generated with data
      ↓
Sent to browser

🧠 Difference from client-side
Client-side
Page loads first
Then fetch data

Server-side
Fetch data first
Then send page

## 🔥 In Pages Router
Use:
getServerSideProps()

🚀 Basic Example
export async function getServerSideProps() {
  const res = await fetch(
    "https://jsonplaceholder.typicode.com/users"
  )

  const data = await res.json()

  return {
    props: {
      users: data
    }
  }
}
export default function Users({ users }) {
  return (
    <div>
      {users.map((user) => (
        <p key={user.id}>{user.name}</p>
      ))}
    </div>
  )
}

### 🧠 What happens?
Step 1
User visits page.

Step 2
Next.js server runs:
getServerSideProps()

Step 3
Server fetches API data.

Step 4
Page rendered WITH data already.

Step 5
Browser receives ready HTML.

🔥 Biggest Advantage
Better SEO
Because HTML already contains data.

Good for:
blogs
public pages
search engines

⚡ Another Advantage
No loading spinner initially.

Because data already available.

💡 Important Rule
getServerSideProps()
runs ONLY on server.
Never in browser.

🧠 Quick Comparison
Client-side	        Server-side
Fetch after render	Fetch before render
Uses useEffect	    Uses getServerSideProps
More interactive	Better SEO
Loading states	    Ready HTML

## MOST important tradeoff of server-side rendering

🧠 What actually happens in SSR

With:
getServerSideProps()

flow becomes:
Request page
   ↓
Server waits for API
   ↓
Generate HTML
   ↓
Send page

👉 So YES:

page response waits for API
slower API = slower page response

🚀 Then why use SSR?

Because browser receives:
READY HTML WITH DATA

instead of:
Empty page + loading spinner

🔥 Main Tradeoff
Client-side fetching
Fast initial page shell

BUT:
data comes later
loading state visible
Server-side fetching
Slower initial response

BUT:
complete page arrives ready

🧠 Visual Difference
Client-side
Page appears fast
Loading...
Then data appears

SSR
Wait slightly longer
Then full page appears

💡 Why companies still use SSR
Because for:
SEO
blogs
e-commerce
public content

Google/search engines need:
HTML with actual content

🔥 Modern Next.js tries to balance both
Using:
static rendering
SSR
client fetching
streaming
caching

depending on use case.

### 👉 SSR hides API request flow
NOT the final data itself.

Client-side fetch
Browser → API
API visible in Network tab ✅

SSR
Browser → Next.js server → API
API usually NOT visible in browser Network tab ✅