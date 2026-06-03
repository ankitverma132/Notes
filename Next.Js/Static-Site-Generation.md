# **Static Site Generation**
Meaning:

HTML is generated at BUILD TIME
not on every request.

🧠 Flow
Build app
   ↓
Fetch data once
   ↓
Generate HTML files
   ↓
Serve ready static files
🔥 Different from SSR
SSR
Every request:
Request → Fetch API → Generate HTML

SSG
Only during build:
Build → Fetch API → Generate HTML once
Then pages are reused.

🚀 In Pages Router
Use:
getStaticProps()

Basic Example
export async function getStaticProps() {
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

🧠 When does API call happen?
NOT when user opens page.
ONLY during:
npm run build

🔥 Then what user gets?
Already-generated static HTML file.
VERY fast ⚡

🚀 Biggest Advantage
Performance
Because no runtime fetching needed.

⚡ Also good for SEO
Since HTML already contains content.

💡 Best Use Cases
Perfect for:
blogs
docs
marketing pages
portfolios
public static content
⚠️ Bad for frequently changing data

Because page becomes stale after build.

Example:
stock prices
live dashboard
realtime analytics


## 🚀 Dynamic Routes with SSG
Use:

getStaticPaths()
along with:
getStaticProps()

Example
pages/posts/[id].js

Need to tell Next.js:
which ids to prebuild

using:
getStaticPaths()

🚀 Problem
For static generation:

getStaticProps()

Runs at build time.
But route is dynamic:

/posts/1
/posts/2
/posts/3

How will Next.js know which pages to generate?

🔥 Solution
Use:
getStaticPaths()

🧠 Responsibilities
getStaticPaths()

Tells Next.js:
which dynamic pages to build

getStaticProps()
Fetches data for EACH page.

### 🚀 Full Example
export async function getStaticPaths() {
  return {
    paths: [
      { params: { id: "1" } },
      { params: { id: "2" } },
      { params: { id: "3" } }
    ],
    fallback: false
  }
}

export async function getStaticProps(context) {
  const { id } = context.params
  const res = await fetch(
    `https://jsonplaceholder.typicode.com/posts/${id}`
  )

  const post = await res.json()

  return {
    props: {
      post
    }
  }
}

export default function Post({ post }) {
  return <h1>{post.title}</h1>
}

🧠 What happens during build?
Step 1

getStaticPaths() runs:

1
2
3
Step 2

Next.js creates:

/posts/1
/posts/2
/posts/3
Step 3

For EACH route:

getStaticProps()

runs separately.

🚀 Important Part
params
params: { id: "1" }

maps to:
/posts/1

⚡ fallback
fallback: false

Only generated routes exist.
Example:
/posts/999

👉 404 page.

### We can call APIs in getStaticPaths.
In fact:
👉 getStaticPaths() commonly calls APIs.

Because it needs to know:

which dynamic routes should be generated.

🚀 Example
Suppose backend gives all post IDs:
GET /posts
Then inside getStaticPaths()
export async function getStaticPaths() {
  const res = await fetch(
    "https://jsonplaceholder.typicode.com/posts"
  )

  const posts = await res.json()

  const paths = posts.map((post) => ({
    params: {
      id: post.id.toString()
    }
  }))

  return {
    paths,
    fallback: false
  }
}

🧠 What this API call does
It fetches:
all available post ids

Then Next.js knows:
/posts/1
/posts/2
/posts/3

should be pre-generated.

🔥 Then getStaticProps() runs

For EACH path separately.
Example:

export async function getStaticProps(context) {
  const { id } = context.params

  const res = await fetch(
    `https://jsonplaceholder.typicode.com/posts/${id}`
  )
  const post = await res.json()

  return {
    props: {
      post
    }
  }
}

🚀 Build Flow
getStaticPaths()
   ↓
Get all ids
   ↓
Generate routes
   ↓
getStaticProps() per route
   ↓
Generate static HTML

getStaticPaths()

fetches:

all blog slugs
getStaticProps()

fetches:
individual blog content

⚠️ Important

Both:
getStaticPaths()
getStaticProps()

run ONLY during:
build time
NOT browser.