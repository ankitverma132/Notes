# **Image Component**

Image component in Next.js is an optimized replacement for normal HTML <img> tag 🔥

Import:
import Image from "next/image"
🚀 Basic Example
import Image from "next/image"

export default function Home() {
  return (
    <Image
      src="/gym.jpg"
      width={300}
      height={200}
      alt="Gym"
    />
  )
}

🧠 Why use Next.js Image instead of <img>?
Because Next.js automatically:
optimizes images
lazy loads
resizes
improves performance

🔥 Main Benefits
Feature	            Benefit
Lazy loading	    Faster page
Optimization	    Smaller image size
Responsive loading	Different image sizes
Better performance	Improves Lighthouse score

🚀 Local Images
Place image inside:
public/

Example:
public/gym.jpg

Use:
src="/gym.jpg"

🚀 Remote Images
<Image
  src="https://example.com/image.jpg"
  width={300}
  height={200}
  alt="Image"
/>

**⚠️ For remote images**
Need to allow domain in:
next.config.js

Example:
module.exports = {
  images: {
    domains: ["example.com"]
  }
}

🔥 Important Props
Prop	Meaning
src	    image path
width	width
height	height
alt	    accessibility text

🧠 Difference from <img>
Normal HTML
<img src="/gym.jpg" />

No optimization.

Next.js Image
<Image src="/gym.jpg" />

Optimized automatically.
⚡ Lazy Loading

Image loads only when near viewport.
Huge performance boost.
Automatic in Next.js.

⚠️ Important
width and height usually required.
Because Next.js wants to prevent:

layout shift


🚀 Normal HTML Image
<img src="/gym.jpg" />

Browser downloads FULL image directly.

Problems:

maybe huge size
loads immediately
wastes bandwidth
slower page
🔥 Next.js Image Component
<Image
  src="/gym.jpg"
  width={300}
  height={200}
  alt="Gym"
/>

## Benifit over <img>
Next.js does smart things automatically.

🧠 1. Lazy Loading
Normal <img>
ALL images load immediately.
Even images below screen.

Example
Suppose page has:
50 workout images
Browser starts downloading ALL 50 😵

Slow.

Next.js Image
Loads image ONLY when near viewport.
Meaning:
image below screen waits
downloads later when user scrolls

Huge performance boost.

🚀 2. Image Resizing
Suppose original image:
4000px × 3000px

But UI only needs:
300px × 200px

Normal <img>
Still downloads HUGE image 😵
Next.js Image
Automatically serves smaller optimized version.

Less MB downloaded ⚡

🚀 3. Modern Formats

Next.js can convert images into:
WebP
AVIF

These are smaller than:
jpg
png
Example

Original:
2 MB jpg

Optimized:
300 KB WebP

Same visual quality almost.

🚀 4. Prevent Layout Shift
Without width/height:
Page jumps while image loads 😵

Bad UX.

Example
Text suddenly moves down after image appears.

Next.js Image
Knows image dimensions beforehand.
So browser reserves space already.

Stable layout ✅

🚀 5. Responsive Images
Different devices get different image sizes.

Example
Mobile

Gets:
small image
Desktop

Gets:
larger image

Saves bandwidth.

🔥 Real-world Effect
Without optimization:

slow loading
high bandwidth
poor Lighthouse score

With Next.js Image:
faster loading
less data usage
smoother UX

## What is loader in Next.js Image?

It is just:
a function that creates image URL.

That’s it.
🧠 Normal Case
<Image src="/gym.jpg" />

Next.js itself decides:
what final image URL should be
🔥 With loader

YOU decide final URL.

Example
const myLoader = ({ src, width }) => {
  return `${src}?w=${width}`
}

Then
<Image
  loader={myLoader}
  src="/gym.jpg"
  width={300}
  height={200}
  alt="Gym"
/>

🧠 Internally
Loader receives:

src = "/gym.jpg"
width = 300
and returns:
/gym.jpg?w=300

🚀 Why needed?
Because some image servers/CDNs understand URL params like:
?w=300

Meaning:
give resized image.

🚀 Without loader

Next.js controls image URL generation + optimization.

<Image src="/gym.jpg" />

Next.js internally creates optimized URL like:

/_next/image?url=/gym.jpg&w=384&q=75

It handles:

resizing
compression
optimization

automatically.

🚀 With loader

YOU control final URL generation.

loader={({ src, width }) =>
  `https://cdn.com/${src}?w=${width}`
}

Now Next.js says:

okay bro you handle image URL yourself 😄

🧠 Real Difference
Without Loader	    With Loader
Next.js optimizes	External service/CDN optimizes
Next.js builds URL	You build URL
Easier	            More customizable
Default behavior	Custom behavior

🔥 Visual Understanding
Without loader
Browser
   ↓
Next.js image optimizer
   ↓
Image
With loader
Browser
   ↓
Your custom CDN/server
   ↓
Image

🚀 Example

Suppose company uses:
Cloudinary
AWS CDN
Imgix

These already optimize images.
So using Next.js optimizer again may be unnecessary.

Then loader tells:

use THIS CDN URL instead.