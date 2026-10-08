---
title: "Client vs SSR, CSR, and Loading State"
parent_post: FocusTube
module_number: 6
layout: module
media_subpath: /assets/tutorials/focustube
---

## Client vs SSR

React is a primarily "Client Side Library". What does this mean?

- Client Side Rendering means the browser receives basic HTML page with links to the JS files. Basically, **its running purely on your browser**.

NextJS has both Client side rendering, as well as Server Side Rendering

**What is Server Side Rendering?**

- Server Side Rendering (SSR) is the process of **rendering the webpage fully on the server** and then sending the fully rendered HTML to the client

By default, NextJS is SSR. However, we can easily make a file client side with this:

`'use client'`

### When do I use Client vs SSR?

There are some times you CANNOT use client side. Your code should be SSR if you meet any of this criteria:

- You need fresh data in every response
- It is important for the page to be fully rendered when displayed
- You access server-only secrets or credentials (ie. calling your internal API for data)

To choose which to use:

**If you need to access to external data or something confidential before the page loads, use SSR**

**If the data can be fetched after the page loads or you are making an interactive page with UI updates (ie. useState()), use CSR.**

> QUESTION: Think of an example use case for both SSR and CSR
{: .prompt-tip }



## Implementing Client Side Rendering

We are going to have **2 components that use client side rendering**

- `app/video/[videoId]/page.js`
- `app/page.js`

**Why make these two Client Side?**

Web pages loading before everything is ready is fine, whereas showing incomplete results while in a search is not ideal.

### Challenge Task

> Make these two components client side components. When doing this, you will run into a couple errors. Attempt to Google or look at the documentartion for why you get these errors.
{: .prompt-warning }

The solution will be provided for `app/video/[videoId]/page.js`. Feel free to take a look if you get stuck or when you finish to ensure are working in the right direction.

The solution for `app/page.js` won't be provided since it will not throw any errors when making it a Client Side Component.

```jsx
"use client";

import { useParams } from "next/navigation";


export default function Content() {


 // since this is NOT a server side component we CANNOT use await
 // luckily NextJS provides a useParams hook to get the params from the URL
 // this is a client side component this is our equivalent to the server side
 // of 'await params'
 const { id } = useParams();


  return (
   <div className="h-lvh w-lvw flex flex-col items-center justify-start text-white">
       <iframe 
         width="960"
         height="540"
         // we can add the id directly to the src
         src={`https://www.youtube.com/embed/${id}`}
         title="YouTube video player"
         frameBorder="0"
         allowFullScreen
         className="rounded-lg shadow-lg"
       >
       </iframe>
   </div>
 );
}
```
{: file="app/video/page.js" }
{: .nolineno }
{: .blur }

> Since our main page (`app/page.js`) is meant to route us to other pages and is a CSR, use `useRouter()` to navigate the buttons to its corresponding pages.
{: .prompt-warning }
{: .blur }

## Loading State

When you have Service Side Components, the entire webpage renders **on the server**. The main problem with this is: *If the content is not ready, what do we show the user?*

NextJS has a very simple and easy way to solve this issue; you can make a **static loading page**.

- If you have a SSR Component, in the **same folder**, make a `loading.js` file
- This `loading.js` file needs to be **static** so they can be ready right when someone loads the page.

> QUESTION: As you might be picking up on, NextJS has some very useful file naming conventions. Can you think of any other useful files you may want along with your name page? Cross compare with the documentation and track if they were on there. https://nextjs.org/docs/app/api-reference/file-conventions
{: .prompt-tip }

This is what your file tree should look like.

```
my-app/
├── node_modules/
├── public/
├── src/
│   └── app/
│       ├── favicon.ico
│       ├── globals.css
│       ├── layout.js
│       ├── page.js
│       ├── search/
│       │   └── [searchId]/
│       │       ├── page.js
│       │       └── loading.js       
│       ├── video/
│       │   └── [videoId]/
│       │       └── page.js          
│       └── playlist/
│           └── [playlistId]/
│               ├── page.js
│               └── loading.js       
├── .gitignore
├── eslint.config.mjs
├── jsconfig.json
├── next.config.mjs
├── package-lock.json
├── package.json
├── postcss.config.js
└── README.md
```

Feel free to make your own `loading.js`, but if you don't want to, here is what mine looks like:

```jsx
export default function Loading() {
   return (
       <div className="flex w-lvw h-lvh justify-center content-center">
           <h1>Loading...</h1>
       </div>
   );
}
```

## Completion & Discussion Checklist

Before joining the group discussion or concluding this tutorial, ensure you have completed the tasks, investigated the bugs, and are ready to discuss the questions below:

<details markdown="1">
<summary>Click to expand Completion & Discussion Checklist (14 Items)</summary>

| # | Type | Item | Prompt Preview |
| :-: | :--- | :--- | :--- |
| 1 | Bug Hunt | Mistyped `<iframe>` Attribute | In the iframe solution, one of the JSX attributes is deliberately mistyped. Use the editor's Problems panel to identify the invalid property and fix it. |
| 2 | Question | Tooling & Router Paradigms | What exactly is ESLint? What is the architectural difference between the Next.js App Router and the older Pages Router? |
| 3 | Question | Project Config Files | What are `.gitignore`, `node_modules`, and `package.json` for? What happens if you delete `package-lock.json`? |
| 4 | Question | Component Definition | What is a React component? Learn a high-level definition to understand how components modularize UI code. |
| 5 | Question | Component Return Values | What do React component functions return under the hood? |
| 6 | Question | `<Link>` vs. Standard `<a>` Anchor | Why do we import the `<Link>` component instead of using a standard `<a>` tag? What happens in the browser if you use a standard anchor? |
| 7 | Question | Input Events & `e.target.value` | In `onChange={(e) => setInput(e.target.value)}`, what is `e.target.value` and where does it come from? |
| 8 | Question | `"use client"` Boundary | What does `"use client";` at the top of a file do? Why do interactive hooks like `useState` require it? |
| 9 | Question | HTML Form Semantics | What is a `<form>` element and how does it change how we structure input handling? |
| 10 | Question | SSR vs. CSR Trade-offs | Think of an example use case for both Server-Side Rendering (SSR) and Client-Side Rendering (CSR). When would you prefer one over the other? |
| 11 | Question | App Router File Conventions | Next.js uses specific file conventions (`layout.js`, `loading.js`, `error.js`). Can you identify other special files in the documentation? |
| 12 | Task | Create Playlist Route | Make the playlist route inside `/app/playlist/page.js`. Create a component that shows mock video cards and links each one to `/video`. |
| 13 | Challenge | Dynamic Video Route Binding | Connect the route parameter in `/app/video/[videoId]/page.js` to the embedded `<iframe>` `src` to play dynamic video IDs. |
| 14 | Challenge | Static Loading Boundary | Create a static `loading.js` component inside your route directory to render fallback UI during async route transitions. |

</details>