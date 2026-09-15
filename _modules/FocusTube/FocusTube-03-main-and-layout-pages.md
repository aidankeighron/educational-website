---
title: "Main and Layout Pages"
parent_post: FocusTube
module_number: 3
layout: module
media_subpath: /assets/tutorials/focustube
---

## Main and Layout Pages

### Main Page

Our main page will also be relatively simple. There are just a couple things to implement, all of which you should be able to do on your own. Working code will be included below, but make sure to attempt it yourself first.

**What the main page should look like:**
- A search bar, in which you can type anything
   - Searching should not be possible when the input is blank
- 4 different buttons, each leading to 4 different types of searches
   - **Regular Search Button:** A regular video search
   - **Playlist Search Button:** A search for playlists specifically
   - **Video ID Button:** Instead of a search, if you know the video ID, type it here and it will go straight to the video
   - **Playlist ID Button:** Like the video ID button, this will take you directly to the playlist instead of searching for it

### Understanding React State (`useState`)

Before we build the search bar, we need a way to keep track of what the user is typing into it. In React, regular variables don't work for this because updating them doesn't tell the page to re-render and show the new text. 

Instead, we use a React Hook called `useState`. It allows us to create a special variable (our state) and a function to update it. When we use the update function, React knows the state has changed and automatically updates our webpage to reflect the new data!

Here is how we use it to track an input:

```jsx
import { useState } from 'react';

export default function MyComponent() {
  // We initialize our state variable 'text' to an empty string. 
  // 'setText' is the function we will call to update it.
  const [text, setText] = useState("");

  return (
    <input 
      value={text} 
      onChange={(e) => setText(e.target.value)} 
    />
  );
}
```

> **QUESTION:** In the `onChange` event above, what is `e.target.value` and where does it come from?
{: .prompt-tip }

### Handling Button Clicks

To make our navigation buttons work, we will also need the `onClick()` event handler. Let's see how that combines with navigation!

<!-- START NEW USEROUTER SECTION -->
### Using NextJS `useRouter` for Navigation

While `Link` is great for simple buttons and images, sometimes you need to navigate to a page after some logic runs—like when a user submits a search form! 

For programmatic navigation, Next.js provides the `useRouter` hook. It's basically NextJS's `<a>` tag for javascript functions.

```jsx
import { useRouter } from 'next/navigation';

export default function MyComponent() {
  const router = useRouter();

  const handleAction = () => {
    // Sends you to another route in your website without reloading the page!
    router.push('/specified/path');

    // You can also refresh the current route you are at
    // router.refresh();

    // Or go back in your browser history
    // router.back();
  };
  
  // ...
}
```
<!-- END NEW USEROUTER SECTION -->


**Click below to unblur the different parts of the answer**

**Part 1: State and Navigation Setup**
First, we need to set up our React state to keep track of what the user types into the search bar. We also initialize `useRouter()` so we can send the user to different pages when they click the buttons.
```jsx
"use client";

import React, { useState } from "react";
import { useRouter } from "next/navigation";

export default function Home() {
 const [input, changeInput] = useState("");
 const router = useRouter();

 const handleInputChange = (e) => {
   changeInput(e.target.value);
 };

 const handleSubmit = (e) => {
   e.preventDefault();
   if (input.trim() !== "") {
     router.push(`/search/${input}`);
   }
 };
 
 // ... return statement below ...
```
{: file="app/page.js (Part 1)" }
{: .nolineno }
{: .blur }

**Part 2: The Search Form**
Next, we build the search form. We tie the `<input>` value to our state, and when the user presses Enter (triggering `onSubmit`), we prevent the default page reload and use `router.push()` to navigate to the search results!

Please don't skip through without giving it a solid attempt :)

```jsx
 // ... inside Home() return statement ...
 return (
   <div>
     <div>
       <h1>Focus</h1>
       <h1>Tube</h1>
     </div>
     <div>
       <form onSubmit={handleSubmit}>
         <input
           value={input}
           onChange={handleInputChange}
           type="text"
           placeholder="Search for something..."
         />
         <button type="submit">Search</button>
       </form>
     </div>
     {/* ... navigation buttons below ... */}
```
{: file="app/page.js (Part 2)" }
{: .nolineno }
{: .blur }

**Part 3: The Navigation Buttons**
Finally, we add our quick-navigation buttons. Since these don't require a form submission, we can just use simple `onClick` events to check if the input is empty, and if not, route the user to the correct page.
```jsx
     {/* ... inside Home() return statement ... */}
     <div>
       <button
         onClick={() => {
           if (input.trim() !== "") {
             router.push(`/playlist-search/${input}`);
           }
         }}
       >
         Search Playlists
       </button>
       <button
         onClick={() => {
           if (input.trim() !== "") {
             router.push(`/playlists/${input}`);
           }
         }}
       >
         Search Playlist ID
       </button>
       <button
         onClick={() => {
           if (input.trim() !== "") {
             router.push(`/video/${input}`);
           }
         }}
       >
         Search Video ID
       </button>
     </div>
   </div>
 );
}
```
{: file="app/page.js (Part 3)" }
{: .nolineno }
{: .blur }

> **QUESTION:** We had to add `"use client";` to the top of our `app/page.js` file, otherwise NextJS would crash. What does `"use client";` do, and why do React hooks like `useState` require it?
{: .prompt-tip }

### Layout Page

This page is unique in NextJS, but is also very powerful.

**The layout page is the UI that is shared between routes.**

What does that mean specifically? Well, have you noticed that every single part of the web app has had the same colors and font? That is thanks to ```layout.js```.

No matter where you are in the website, **what is in the layout page is applied there as well**.

This often includes:

- Any CSS you want to add
- Headers and/or footers
- A navigation bar for the entire website

Feel free to make your Layout page your own. Luckily, NextJS already has a really useful one made for you that you can tweak!
