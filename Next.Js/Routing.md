# **Routing**

Structure:

pages/
 ├── index.js
 ├── about.js
 └── dashboard.js

Routes:
File	        Route
index.js	    /
about.js	    /about
dashboard.js	/dashboard

🔥 Nested Routes

Example:
pages/
 └── dashboard/
       └── settings.js

Route:
/dashboard/settings

## Default export
Default export in JavaScript/React basically means:
“This is the MAIN thing this file exports”

🧠 Simple Example
export default function Home() {
  return <h1>Hello</h1>
}

👉 Here:
Home is exported as default
Other files can import it easily

⚡ Why “default”?
Because a file can have:
ONE default export only

### 🧱 Named Export vs Default Export
✅ Default Export
export default function Button() {}

Import:
import Button from "./Button"

✅ Named Export
export function Button() {}

Import:
import { Button } from "./Button"

👉 {} required

🔥 Main Difference
Default Export	        Named Export
One per file	        Multiple allowed
No {}	                Uses {}
Can rename on import	Same name needed

🧠 Example of Renaming
Default export:
import AnythingName from "./Button"

Works because:
👉 Default export can be renamed freely.

Named export:
import { Button } from "./Button"

Must match name.

🚀 In Next.js
You’ll see this everywhere:
export default function Home() {
  return <div>Home Page</div>
}

Because:
👉 Next.js expects the page component as default export.

## For route pages, Next.js expects a default export. If you only use named export, route will usually fail/error.

🧠 Example (Correct)
export default function Home() {
  return <h1>Home</h1>
}

✅ Works for:
pages/index.js

❌ Example (Wrong for route pages)
export function Home() {
  return <h1>Home</h1>
}

👉 Here there is NO default export.
🔥 Why?
Because internally Next.js does something conceptually like:

import Page from "./index"

🧩 BUT named exports are still allowed INSIDE page files
Example:
export function helper() {}

export default function Home() {
  return <h1>Home</h1>
}

✅ This works.
Because:
Home = default export
helper = additional named export

### 🧠 Why use named exports inside page files?

Because page files can contain:
helper functions
utility methods
constants
configs

along with the main page component.

🚀 Example
export function calculateBMI(weight, height) {
  return weight / (height * height)
}

export default function Home() {
  return <h1>Health App</h1>
}

## Dynamic Routes
Dynamic routes in Next.js are used when:
👉 route value changes dynamically

Example:
Product ID
User ID
Blog slug

Instead of creating:
/workout1
/workout2
/workout3

You create ONE dynamic route:
/workouts/[id]

🚀 Example Structure (Pages Router)
Since you’re using old routing:

pages/
 └── workouts/
       └── [id].js

🧠 What URLs will this match?
/workouts/1
/workouts/abc
/workouts/chest-day

👉 All handled by same file:
[id].js

🔥 Why [id].js?
Square brackets mean:
“This part is dynamic”

⚡ Accessing the dynamic value
Use:
import { useRouter } from "next/router"
Example
import { useRouter } from "next/router"

export default function WorkoutPage() {
  const router = useRouter()

  return <h1>Workout ID: {router.query.id}</h1>
}

🧠 If URL is:
/workouts/25
Then:
router.query.id
becomes:
25

🧩 Another Example
pages/
 └── users/
       └── [username].js

URLs:
/users/ankit
/users/john

🔥 Catch-All Dynamic Routes
For multiple segments:
pages/
 └── docs/
       └── [...slug].js

Matches:
/docs/react
/docs/react/hooks
/docs/react/hooks/useEffect

## UseRoute Hook
🚀 What router hook gives you

It lets you access:
current route
query params
navigation methods

🧠 Example
import { useRouter } from "next/router"
const router = useRouter()

Now router contains route info.

🔥 Dynamic Route Example
File:
pages/workouts/[id].js
URL:

/workouts/25
Code
import { useRouter } from "next/router"

export default function WorkoutPage() {
  const router = useRouter()

  return <h1>{router.query.id}</h1>
}

Output:
25

### 🧩 Router Object Useful Things
🥇 Current route
router.pathname

Example:
/workouts/[id]

🥈 Query params
router.query.id

🥉 Navigate programmatically
router.push("/dashboard")

👉 Redirect user.

⚠️ Important
useRouter() only works:
inside React component
client-side

🚀 Important Router Properties
🥇 router.query

Gets dynamic route params + query params.
Example:

URL:
/workouts/25?type=chest
router.query.id     // 25
router.query.type   // chest

🥈 router.pathname
Gives route pattern.
Example:
File:
pages/workouts/[id].js
Returns:

router.pathname
// "/workouts/[id]"

🥉 router.asPath
Gives actual URL shown in browser.
Example:
/workouts/25?type=chest
Returns:
router.asPath
// "/workouts/25?type=chest"

⚡ router.push()
Navigate programmatically.
router.push("/dashboard")

🔙 router.back()
Go back like browser back button.
router.back()

🔄 router.reload()
Reload current page.

router.reload()

🧩 router.replace()
Like push() but:
👉 doesn’t add history entry.

Useful after login.
router.replace("/dashboard")

User can’t press back to login page easily.

🧠 router.isReady
Very important sometimes.
Because query params may initially be empty.

if (!router.isReady) return null

🔥 Real Example
import { useRouter } from "next/router"

export default function WorkoutPage() {
  const router = useRouter()
  const { id } = router.query
  return (
    <div>
      <h1>Workout {id}</h1>

      <button onClick={() => router.push("/dashboard")}>
        Go Dashboard
      </button>
    </div>
  )
}

💡 Most Used in Real Projects

Property	Usage
query	    params
push()	    navigation
pathname	route info
replace()	redirects
isReady	    safe query access

⚠️ Small Important Thing
Initially:
router.query.id
can be:
undefined
during first render.

That’s why:
router.isReady

sometimes needed.

### Pathname vs as path
The difference is:
pathname → route structure/pattern
asPath → actual browser URL

🧠 Example
Suppose file is:

pages/workouts/[id].js

User opens:
/workouts/25?type=chest
🚀 router.pathname
router.pathname

returns:
/workouts/[id]

👉 It shows:
route definition/pattern
NOT actual value.

🚀 router.asPath
router.asPath

returns:
/workouts/25?type=chest

👉 Actual URL shown in browser.

Including:
dynamic values
query params

### query.name
query.name depends on your dynamic route parameter name 🔥

Example:
🚀 Dynamic Route File
pages/users/[name].js

Here:
[name]

means:
👉 dynamic param is called "name"

🧠 URL
/users/ankit
Code
const router = useRouter()

console.log(router.query.name)
Output
ankit

## 🚀 Dynamic Nested Routing
🚀 Option 1 — [id].js (single file)
pages/
 └── workouts/
       └── [id].js

Handles:
/workouts/25

🚀 Option 2 — [id]/ folder
pages/
 └── workouts/
       └── [id]/
             ├── index.js
             └── edit.js

🧠 What routes get created?
File	    Route
index.js	/workouts/25
edit.js	    /workouts/25/edit

🔥 Why use folder approach?

When that dynamic route has:
subpages
nested routes
more structure

💡 Real Example for YOUR app
pages/
 └── workouts/
       └── [id]/
             ├── index.js
             ├── edit.js
             └── analytics.js

Routes:
/workouts/101
/workouts/101/edit
/workouts/101/analytics

⚡ Accessing query still works same
router.query.id
returns:
101

### Reading query parameter:
In this structure bro 👇

pages/
 └── workouts/
       └── [id]/
             ├── index.js
             └── edit.js

Suppose URL is:

/workouts/25/edit?mode=advanced
🚀 Read Query Params

Inside:
edit.js
do this:

import { useRouter } from "next/router"

export default function EditWorkout() {
  const router = useRouter()

  const { id, mode } = router.query

  return (
    <div>
      <h1>Workout ID: {id}</h1>
      <h2>Mode: {mode}</h2>
    </div>
  )
}

🧠 Values
For URL:
/workouts/25/edit?mode=advanced
You get:
id = "25"
mode = "advanced"

🔥 Important Understanding
router.query contains BOTH:

dynamic route params
query string params

🧩 Example
URL Part	    Access
/25/	        router.query.id
?mode=advanced	router.query.mode

💬 Mental model
Dynamic segment:
[id]

👉 becomes:

router.query.id
Query string:
?mode=advanced

👉 becomes:

router.query.mode

🔥

### router.push with query and pathname
🚀 Syntax
router.push({
  pathname: "/workouts/[id]",
  query: {
    id: 25,
    mode: "advanced"
  }
})

🧠 Generated URL
/workouts/25?mode=advanced

🔥 Full Example
import { useRouter } from "next/router"

export default function Home() {
  const router = useRouter()

  const goToWorkout = () => {
    router.push({
      pathname: "/workouts/[id]/edit",
      query: {
        id: 25,
        mode: "advanced"
      }
    })
  }

  return (
    <button onClick={goToWorkout}>
      Go
    </button>
  )
}

🧠 Result URL
/workouts/25/edit?mode=advanced
⚡ Important Understanding
In:
pathname: "/workouts/[id]"

👉 [id] is replaced using:

query.id
🧩 Another Example
router.push({
  pathname: "/users/[name]",
  query: {
    name: "ankit",
    tab: "profile"
  }
})

URL becomes
/users/ankit?tab=profile

💡 Rule
Params matching dynamic segments:
id
name
slug

👉 become part of path.
Extra query values:
mode
tab
theme

👉 become query string.
💬 Simple mental model
pathname = route structure
query = values for route + URL params

### Diff between router push and router replace

router.push() and router.replace() both navigate to another page, BUT difference is in browser history.

🚀 router.push()

Adds new entry to browser history.

router.push("/dashboard")
🧠 What happens?

History stack:

Login → Dashboard

Now user presses back:
👉 Goes back to Login page.

🚀 router.replace()

Replaces current history entry.

router.replace("/dashboard")
🧠 What happens?

History becomes:

Dashboard

Login page removed from history.

User presses back:
👉 Won’t go back to Login.

🔥 Real-World Usage
✅ Use push()

For normal navigation:

dashboard
profile
workouts

Example:

router.push("/profile")
✅ Use replace()

For redirects/navigation you DON’T want in history.

Examples:

after login
after logout
auth redirects
💡 Login Example
router.replace("/dashboard")

Why?
Because after login:
👉 user shouldn’t go back to login page using back button.

## Catch-All Route

🚀 1. Normal Dynamic Route
[id]
Matches ONE segment only.
Example:
/workouts/25
Then:
router.query.id
// "25"

🚀 2. Catch-All Route
[...tab]
Means:
“Take ALL remaining route parts”

Example Folder
pages/docs/[...tab].js

URLs it matches
/docs/react
/docs/react/hooks
/docs/react/hooks/useEffect

What value comes?
URL:
/docs/react/hooks

Then:
router.query.tab
becomes:
["react", "hooks"]

👉 ARRAY because multiple segments.

⚠️ BUT this will NOT work:
/docs
Because:
[...tab]

requires at least ONE segment.

🔥 NOW THE IMPORTANT ONE
[[...tab]]
Extra [] means:
“optional”
Folder

pages/docs/[[...tab]].js
Now ALL these work:
/docs
/docs/react
/docs/react/hooks

🧠 What values come?
Case 1
URL:
/docs
Then:
router.query.tab
👉
undefined

Because no extra segments.

Case 2
URL:
/docs/react
Then:
router.query.tab

👉
["react"]

Case 3
URL:
/docs/react/hooks
Then:
router.query.tab
👉

["react", "hooks"]

🔥 MOST IMPORTANT DIFFERENCE

Route Type	    /docs works?
[...tab]	    ❌ No
[[...tab]]	    ✅ Yes

🧠 Simple mental model
[id]

One value

[...tab]

Many values REQUIRED

[[...tab]]

Many values OPTIONAL

💡 Real-world analogy

Imagine route:

/docs

can optionally continue with:

/chapter
/topic
/subtopic

👉 That flexibility is why optional catch-all exists.

🚀 Example in YOUR health app

Could use:

pages/reports/[[...filters]].js

URLs:

/reports
/reports/monthly
/reports/monthly/2026


n MANY cases:

[id]/
   index.js
   edit.js

can achieve similar routing behavior to catch-all routes.

BUT the purpose is different.

🚀 Your Structure
pages/
 └── [tab]/
       ├── index.js
       └── edit.js

Routes:

/react
/react/edit
/angular
/angular/edit
🧠 What [tab] does here

It captures ONLY ONE segment.

Example:

/react

Then:

router.query.tab
// "react"
⚠️ But this WON’T work
/react/hooks

Because:

[tab]

captures only ONE level.

🔥 Catch-All Route Difference
[...tab]

Can capture:

/react/hooks/useEffect

Then:

router.query.tab

becomes:

["react", "hooks", "useEffect"]
🚀 Comparison
[tab]

ONE segment only.

[...tab]

Unlimited nested segments.

🧩 Visual Comparison
Route Type	    Matches
[tab]	        /react
[tab]/edit.js	/react/edit
[...tab]	    /react/hooks/useEffect

💡 Important Concept
Your folder structure creates:
fixed nested routes

Example:
/react/edit

But catch-all creates:
flexible unknown depth routes

Example:
/react/a/b/c/d/e

YES bro 🔥 technically you can create deeply nested folders with [tab] and achieve many similar routes.

Example:

pages/
 └── [tab]/
       └── [section]/
             └── [topic].js

Matches:

/react/hooks/useEffect

Then:

router.query.tab      // react
router.query.section  // hooks
router.query.topic    // useEffect
🚀 So why even use [...slug] then?

Because with nested [tab] folders:
👉 structure must be known beforehand.

You must define:

[level1]/[level2]/[level3]

manually.

⚠️ Problem

What if route depth changes?

Example:

/docs/react
/docs/react/hooks
/docs/react/hooks/useEffect
/docs/react/hooks/useEffect/examples

👉 Impossible to predict all levels cleanly.

🔥 Catch-all solves this

Using:

[...slug].js

ONE file handles ALL depths.

🧠 Comparison
Nested Dynamic Folders
[a]/[b]/[c]

✅ Structured
✅ Predictable
❌ Fixed depth

Catch-All
[...slug]

✅ Unlimited depth
✅ Flexible
❌ Less structured

💡 Real-world thinking
Use nested dynamic folders when:
structure is known
depth is fixed

Example:

/user/123/post/99
Use catch-all when:
route depth unknown
flexible content hierarchy

Example:

/docs/react/hooks/useEffect/examples


pages/
 └── [tab]/
       └── [section]/
             └── [topic].js

each dynamic folder name becomes a property inside:

router.query
🚀 Example URL
/react/hooks/useEffect
🧠 Values
router.query.tab
// "react"

router.query.section
// "hooks"

router.query.topic
// "useEffect"
🔥 Full Example
import { useRouter } from "next/router"

export default function TopicPage() {
  const router = useRouter()

  const { tab, section, topic } = router.query

  return (
    <div>
      <h1>{tab}</h1>
      <h2>{section}</h2>
      <h3>{topic}</h3>
    </div>
  )
}
🧠 Output for URL
/react/hooks/useEffect

renders:

react
hooks
useEffect


### Extra [] in [[...tab]]

Bro the extra [] means:

OPTIONAL 🔥

That’s the ONLY reason.

🚀 Compare Carefully
[...tab]

Means:

catch all REQUIRED segments

[[...tab]]

Means:

catch all OPTIONAL segments

🧠 Why extra []?

Think like this:
Normal dynamic route
[id]

captures:
/25
Optional dynamic route idea

Extra brackets:
[[id]]

means:
maybe present, maybe not

Same concept applied to catch-all.

🔥 Example
[...tab]
pages/docs/[...tab].js

Works:

/docs/react
/docs/react/hooks

❌ But NOT:
/docs

Because at least ONE segment required.

🚀 Now Optional Catch-All
[[...tab]]
pages/docs/[[...tab]].js

Now ALL work:
/docs
/docs/react
/docs/react/hooks