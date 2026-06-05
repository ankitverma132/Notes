# **CSS In Next JS**

CSS in Next.js is very flexible bro 🔥

You can use:

normal CSS
CSS Modules
global CSS
styled-jsx
Tailwind CSS
Sass/SCSS

But for now focus on:

Global CSS
CSS Modules

These are core.

🚀 1. Global CSS

Applies styles everywhere.

File
styles/globals.css

Import in
pages/_app.js

import "../styles/globals.css"

export default function App({ Component, pageProps }) {
  return <Component {...pageProps} />
}

Example CSS
body {
  background: black;
  color: white;
}

🧠 Used for:
body styles
resets
common typography
global themes

🔥 2. CSS Modules (MOST IMPORTANT)

Scoped CSS.

Meaning:
👉 styles apply ONLY to that component/page.

File Structure
pages/
   index.js

styles/
   Home.module.css
CSS Module
.title {
  color: red;
}
Import
import styles from "../styles/Home.module.css"

export default function Home() {
  return (
    <h1 className={styles.title}>
      Hello
    </h1>
  )
}
🧠 Why CSS Modules?

Avoids conflicts.

Example:

.title

inside one component
won’t affect another component.

🚀 3. Inline Styles
<h1 style={{ color: "red" }}>
  Hello
</h1>

Less common for large apps.

🚀 4. Styled JSX

Built into Next.js.

export default function Home() {
  return (
    <>
      <h1>Hello</h1>

      <style jsx>{`
        h1 {
          color: red;
        }
      `}</style>
    </>
  )
}
🚀 5. Tailwind CSS

VERY popular now.

Utility-based CSS.

Example:

<h1 className="text-red-500 text-2xl">
  Hello
</h1>


⚠️ Important Rule
Global CSS can ONLY be imported in:

pages/_app.js

But CSS Modules:
*.module.css

can be imported anywhere.

## Module css saves from collision:

Problem in normal CSS:

.title {
  color: red;
}

This class name is GLOBAL.

So another file can also have:

.title {
  color: blue;
}

👉 collision/conflict happens.

🚀 Example Problem
Home.css
.title {
  color: red;
}
Profile.css
.title {
  color: blue;
}
⚠️ Both become global

Browser only sees:

.title

So styles may override each other depending on load order.

🔥 CSS Modules Solution

File:

Home.module.css
CSS
.title {
  color: red;
}
Import
import styles from "./Home.module.css"

<h1 className={styles.title}>
  Hello
</h1>
🧠 What Next.js does internally

It converts class name into UNIQUE generated class.

Example:

Home_title__a1b2c

instead of plain:

.title
Another module becomes
Profile_title__x9y8z
🔥 So no collision possible

Because final browser classes are unique.

🚀 Internally Generated HTML

Instead of:

<h1 class="title">

browser gets something like:

<h1 class="Home_title__a1b2c">
💡 Why this is powerful

You can safely use:

.container
.title
.button

inside every component.

No fear of conflicts.

🧠 Mental model

CSS Modules:

auto namespaces your CSS classes

⚡ Normal CSS
GLOBAL
⚡ CSS Modules
LOCAL TO COMPONENT

🔥 Important

Collision prevention happens:
👉 BETWEEN DIFFERENT MODULE FILES

Example:
File	            Generated Class
Home.module.css	    Home_title__abc
Profile.module.css	Profile_title__xyz

🚀 So sharing one module intentionally is allowed
Actually common pattern.

Example:
Button.module.css

used by multiple button-related components.