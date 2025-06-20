---
title: FocusTube
author: Johnny
date: 2025-5-27 12:00:00 +0800
categories: [JavaScript, FocusTube]
tags: [JavaScript, Medium]
description: In this project you will learn the fundamentals of NextJS and APIs in order to make a youtube clone/wrapper that removes the distracting algorithms that hook you in.
comments: false
pin: true
media: /assets/tutorials/focustube
image: /focus_thumbnail.png
---

## About the project

Welcome to the FocusTube tutorial! Through this project, you will learn the fundamentals of making your own API, calling APIs, and the additional features NextJS has over React.

**What you will learn:**

- How to use and create the fundamental parts of an API (endpoints, requests and responses) in NextJS
- How to connect, fetch, parse, and display API data from YouTube
- The fundamentals of NextJS (Server Side Rendering vs Client Side, Dynamic Routing, Loading States, Routing)
- OPTIONAL: Learn how to use TailwindCSS

**What you will make:**

By the end of this tutorial, you will have a basic full-stack YouTube wrapper that allows you to use YouTube through your own design, such as removing YouTube’s distracting features.

**Further Possibilities:**

There are many avenues you can take with this project. While it will be fully functional by the end of this tutorial, there are plenty of opportunities to add your own features and make this project your own. I will go more in depth on these possibilities later in the tutorial.

## Disclaimer

**This tutorial assumes a couple of things:**

- You have decent knowledge of JavaScript
- You have basic knowledge of React
- You already have the necessary tools installed (Node.js, an IDE like VS Code)

## Setup the project

You will be making your own project instead of forking a repository. Making a NextJS app is incredibly easy. Open a terminal. In the terminal, go to the folder you want your NextJS project to be in, then run this command:

```console
$ npx create-next-app@latest
```

You will be given multiple prompts on how you want your project to be. I included the settings I selected, which you can change if you want, but I will only cover the technologies I have selected.

```console
✔ What is your project named? … my-app
✔ Would you like to use TypeScript? … No
✔ Would you like to use ESLint? … Yes
✔ Would you like to use Tailwind CSS? … Yes
✔ Would you like your code inside a `src/` directory? … Yes
✔ Would you like to use App Router? (recommended) … Yes
✔ Would you like to use Turbopack for `next dev`? … Yes
✔ Would you like to customize the import alias (`@/*` by default)? … Yes
```

Now you will now have a folder for your project. I named mine **/my-app**, as seen above. cd into that folder and run:

```console
$ npm run dev
```

This will start up your app and you will can visit it [here at localhost:3000](http://localhost:3000/)

**NOTE:** When you change your project, this will automatically update the website  for you. There is no need to keep terminating and restarting the program.

Now you are ready to start the tutorial!

Take a look at your file tree. It should look something like this:

```
my-app/
├── node_modules/
├── public/
├── src/
│   └── app/
│       ├── favicon.ico
│       ├── globals.css
│       ├── layout.js
│       └── page.js
├── .gitignore
├── eslint.config.mjs
├── jsconfig.json
├── next.config.mjs
├── package-lock.json
├── package.json
├── postcss.config.js
└── README.md
```

Feel free to check out **page.js** in your app folder, this is the **home webpage** in your NextJS app, which is a good transition into the first topic.

## File-Based Routing

File-Based Routing means that the structure of your files and folders inside the **app/** directory automatically defines the routes in your web app.

Let’s take a look at what this really means.

In your **/app** folder create a folder called *```/video```*. In this folder create a file called **page.js**. In this file, put something really simple like this:

```jsx
export default function Video() {
   return (<h1> This is the Video page! </h1>);
}
```
{: file="app/video/page.js" }
{: .nolineno }

Next, go to [http://localhost:3000/video](http://localhost:3000/video)


Here, you will find the component you just wrote. This because of the File-Based Routing in NextJS; since you made a folder in your **/app** directory and gave it a **page.js**, the web app now treats this *as a route*. In other words, any **folder** in the **/app** directory will become a path in your web app (as long as you have a page.js file inside).

For more information, please feel free to use the [NextJS Documentation](https://nextjs.org/docs/app/building-your-application/routing/route-handlers).

## Adding more Routes

Now that we know what file-based routing is, make **2 more routes**.

- ```/search```
- ```/playlist```

For now, these can display whatever you like.

Here is what your file tree should look like now:

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
│       ├── playlist/
│       │   └── page.js
│       ├── search/
│       │   └── page.js
│       └── video/
│           └── page.js
├── .gitignore
├── eslint.config.mjs
├── jsconfig.json
├── next.config.mjs
├── package-lock.json
├── package.json
├── postcss.config.js
└── README.md
```

You should have **4 routes** in your website: the homepage, ```/search``` page, ```/playlist``` page, and the ```/video``` page.

Now that we have a couple routes, we can start to change how they look. Feel free to use the instructions as a guide to your own design, or copy it if you already feel comfortable in React and JS.

### Video Route

It is pretty easy to add youtube videos into you web app even without any API. On most youtube videos, you can click the **Share** button, and you will see an option called **Embed**, which will give you an ```<iframe>``` component that you can add to your project.

Feel free to chose any video and put it in your ```/video``` route

```jsx
export default function Content() {

 return (
   <div>
     <iframe
       width="960"
       height="540"
       src={`https://www.youtube.com/embed/your-youtube-video-id-here`}
       title="YouTube video player"
       allowfullscreen
     ></iframe>
   </div>
 );
}
```
{: file="app/video/page.js" }
{: .nolineno }

> **In the above answer I purposely mistyped one of the attributes of ```<iframe>```.**
{: .prompt-danger }

Luckily, when you save the file and look at it at the ```/video``` web app route, you will see **a NextJS logo that says issue on the bottom left**.

If you click this logo, it will tell you any errors or warnings you may currently have in your file. This will be a massive help while you are programming. However, it won't catch everything, such as *bugs*! Bugs may not be an error; they just don't do what you intended them to do.

When this happens, make sure you always use **Fn+F12**, which lets you inspect your webpage HTML and CSS. Additionally, if you are stuck or unsure what is going on, you can always use various console functions like ```console.log()```, which is the equivalent to ```print()``` in javascript. **I will add areas for you to try these out later**.

### Search Route

Next up, we have our ```/search``` route.

This may sound counter-intuitive, but the search page will be the search results, not the actual search bar. The search bar will be added on the *home page*, which we will get to shortly.

I want you to challenge your JavaScript skills for this one. Do your best not to look at the answer until you have tried it yourself.

For this page, you should:

- Add a React component in the page.js that renders a fake video card/item or two
- Style each video as a card with a thumbnail, title, and description
- Make sure the cards link to the ```/video``` route (we will get it to link to actual videos later)

**Click below to unblur the answer**

```jsx
export default function SearchPage() {
 return (
   <div>
     <h1>Search Results</h1>
     <div>
       <a href="/video">
         <img
           src="https://i.ytimg.com/vi/abc123/mqdefault.jpg"
           alt="How to Learn JavaScript Fast"
           width={160}
           height={120}
         />
         <div>
           <h2>How to Learn JavaScript Fast</h2>
           <p>A quick guide to getting started with JavaScript.</p>
         </div>
       </a>
     </div>
     <div>
       <a href="/video">
         <img
           src="https://i.ytimg.com/vi/abc123/mqdefault.jpg"
           alt="Master C++ in 3 Hours"
           width={160}
           height={120}
         />
         <div>
           <h2>Master C++ in 3 hours</h2>
           <p>An easy guide to teach you everything to get started with C++</p>
         </div>
       </a>
     </div>
   </div>
 );
}
```
{: file="app/search/page.js" }
{: .nolineno }
{: .blur }

### Playlist Route

This one you will have to do on your own, but it is very similar to the search route.

- Inside ```/playlist/page.js```, create a React component that shows fake video cards/items in a playlist
- Style each video as you want
- Make sure each one links to the ```/video``` route

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

**Some things will need:**
- ```useState()``` - familarize yourself with what useState does, as it is very important for the search bar
- ```onClick()``` - figure out how to use this to get to other pages

Do your best to complete this on your own, only using the following code if you get absolutely stuck. You should also avoid the use of AI, but other resources such as **Stack Overflow, Geeks4Geeks, Reddit**, and most powerful of all, **Googling it**, is encouraged. I guarantee you someone has ran into the same issues before.

```jsx
import React, { useState } from "react";


export default function Home() {
 const [input, changeInput] = useState("");

 const handleInputChange = (e) => {
   changeInput(e.target.value);
 };

 const handleSubmit = (e) => {
   e.preventDefault();
   if (input.trim() !== "") {
     window.location.href = `/search/${input}`;
   }
 };

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
     <div>
       <button
         onClick={() => {
           if (input.trim() !== "") {
             window.location.href = `/playlist-search/${input}`;
           }
         }}
       >
         Search Playlists
       </button>
       <button
         onClick={() => {
           if (input.trim() !== "") {
             window.location.href = `/playlists/${input}`;
           }
         }}
       >
         Search Playlist ID
       </button>
       <button
         onClick={() => {
           if (input.trim() !== "") {
             window.location.href = `/video/${input}`;
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
{: file="app/page.js" }
{: .nolineno }
{: .blur }

### Layout Page

This page is unique in NextJS, but is also very powerful.

**The layout page is the UI that is shared between routes.**

What does that mean specifically? Well, have you noticed that every single part of the web app has had the same colors and fontd? That is thanks to ```layout.js```.

No matter where you are in the website, **what is in the layout page is applied there as well**.

This often includes:

- Any CSS you want to add
- Headers and/or footers
- A navigation bar for the entire website

Feel free to make your Layout page your own. Luckily, NextJS already has a really useful one made for you that you can tweak!

## OPTIONAL: TailwindCSS

This portion is optional because I want this tutorial to focus more on APIs and NextJS. However, if you are interested in learning more about TailwindCSS, you can try it out on this project! If not, you are free to use the provided TailwindCSS classes if you would like to change the appearance of your web app.

TailwindCSS is a different way of doing CSS. There are no ```.css``` files; instead, you add classes for each style you want.

For example, in regular CSS, you may make a ```.css``` file then add the class to your HTML like this:

```css
.center-div {
   display: flex;
   justify-content: center;
   align-items: center;
}
```

```html
<div className="center-div"> Hello World </div>
```

In TailwindCSS, each one of these attributes is its own class that we can add straight to the HTML. For example:

```html
<div className="flex justify-center items-center"> Hello! </div>
```

For a deeper dive, I highly suggest Fireship's 100-second video on TailwindCSS.

{% include embed/youtube.html id='mr15Xzb1Ook' %}

### What Mine Looks Like

Feel free to copy this TailwindCSS if you don't want to spend the tutorial fighting CSS. However, I **encourage** you to make this your own and make it look how you want it to.

#### app.js

```jsx
import React, { useState } from "react";


export default function Home() {
 const [input, changeInput] = useState("");


 const handleInputChange = (e) => {
   changeInput(e.target.value);
 };


 const handleSubmit = (e) => {
   e.preventDefault();
   if (input.trim() !== "") {
     window.location.href = `/search/${input}`;
   }
 };


 return (
   <div className="w-screen h-screen flex items-center justify-center flex-col">
     <div className="flex m-3 p-3">
       <div className="w-fit h-fit flex items-center justify-center p-1">
         <h1 className="italic text-white text-4xl">Focus</h1>
       </div>
       <div className="bg-red-800 flex items-center justify-center p-1 rounded-lg">
         <h1 className="text-white text-4xl">Tube</h1>
       </div>
     </div>


     <div className="mt-6 w-full flex justify-center">
       <form onSubmit={handleSubmit} className="w-full max-w-2xl px-4 flex gap-2">
         <input
           value={input}
           onChange={handleInputChange}
           type="text"
           className="bg-neutral-700 text-white pl-4 pr-4 py-3 rounded-full w-full text-lg outline-none focus:ring-2 focus:ring-red-800"
           placeholder="Search for something..."
         />
         <button
           type="submit"
           className="bg-red-800 text-white px-6 py-2 rounded-full text-lg hover:bg-red-700 active:bg-red-900 transition-colors duration-150 cursor-pointer"
         >
           Search
         </button>
       </form>
     </div>


     <div className="mt-10 flex flex-wrap justify-center gap-4">
       <button
         onClick={() => {
           if (input.trim() !== "") {
             window.location.href = `/playlist-search/${input}`;
           }
         }}
         className="bg-neutral-600 text-white px-4 py-2 rounded-2xl hover:bg-neutral-500 active:bg-neutral-700 transition"
       >
         Search Playlists
       </button>


       <button
         onClick={() => {
           if (input.trim() !== "") {
             window.location.href = `/playlists/${input}`;
           }
         }}
         className="bg-neutral-600 text-white px-4 py-2 rounded-2xl hover:bg-neutral-500 active:bg-neutral-700 transition"
       >
         Search Playlist ID
       </button>


       <button
         onClick={() => {
           if (input.trim() !== "") {
             window.location.href = `/video/${input}`;
           }
         }}
         className="bg-neutral-600 text-white px-4 py-2 rounded-2xl hover:bg-neutral-500 active:bg-neutral-700 transition"
       >
         Search Video ID
       </button>
     </div>
   </div>
 );
}
```
{: file="app/page.js" }
{: .nolineno }
{: .blur }

#### Search Page

```jsx
export default function SearchPage() {
 return (
   <div className="w-screen min-h-screen flex flex-col items-center justify-start text-white">
     <h1 className="text-4xl mt-6 mb-4">Search Results</h1>


     <div className="flex flex-col items-center w-full overflow-y-auto pb-10">
       <a href="/video" className="w-1/2 max-w-3xl bg-neutral-700 rounded-2xl p-3 m-3 flex items-start gap-3">
         <img
           src="https://i.ytimg.com/vi/abc123/mqdefault.jpg"
           alt="How to Learn JavaScript Fast"
           width={160}
           height={120}
           className="rounded-md shrink-0 object-fill"
         />
         <div className="flex flex-col justify-start h-full p-3">
           <h2 className="text-lg font-semibold leading-tight mb-1"> How to Learn JavaScript Fast</h2>
           <p className="text-sm text-gray-300 leading-snug">A quick guide to getting started with JavaScript.</p>
         </div>
       </a>


       <a href="/video" className="w-1/2 max-w-3xl bg-neutral-700 rounded-2xl p-3 m-3 flex items-start gap-3">
         <img
           src="https://i.ytimg.com/vi/abc123/mqdefault.jpg"
           alt="Master C++ in 3 Hours"
           width={160}
           height={120}
           className="rounded-md shrink-0 object-fill"
         />
         <div className="flex flex-col justify-start h-full p-3">
           <h2 className="text-lg font-semibold leading-tight mb-1">Master C++ in 3 Hours</h2>
           <p className="text-sm text-gray-300 leading-snug"> An easy guide to teach you everything to get started with C++.</p>
         </div>
       </a>
     </div>
   </div>
 );
}
```
{: file="app/search/page.js" }
{: .nolineno }
{: .blur }

#### Video Page

```jsx
export default function Content() {
 return (
   <div className="h-lvh w-lvw flex flex-col items-center justify-start text-white">
     <iframe
       width="960"
       height="540"
       src="https://www.youtube.com/embed/your-youtube-video-id-here"
       title="YouTube video player"
       frameBorder="0"
       allowFullScreen
       className="rounded-lg shadow-lg"
     ></iframe>
   </div>
 );
}
```
{: file="app/video/page.js" }
{: .nolineno }
{: .blur }

#### Playlist Page

Since you previously implemented the playlist page on your own, I will not be providing code here. Feel free to copy from the search page or make it on your own.

## Dynamic Routing

For many websites, it is impossible to hardcode every single page. This is one such case, as it needs to be able to render **any video on youtube**.

Luckily, NextJS has a pretty simple way to make the webpage differ based on the url provided. It's called Dynamic Routing.

**Here is an example:**

I have this route in my app:

`app/video/page.js`

Currently, this is kind of useless because there is only one video. However, with dynamic routing, we can insert the Video ID into the url to affect the webpage.

In order to implement this, you will need to create a new folder inside your route folder. It should look like this:

`app/route/[routeId]/page.js`

NextJS only knows you have a dynamic route in the folders with brackets around them.

**Pay attention to what you name the folder inside the brackets. That is what we will need to call in the next section.**

### Where we need Dynamic Routing

We need dynamic routing pretty much anywhere the webpage is impacted by the url parameters, which happens to be **all of our routes** in this case.

So, you should:
- Make a new folder in all routes *(/search, /video, /playlist)*
- Name your new folders whatever you want, it just has to be in brackets to work: `[name]`
- Move the **page.js** file in each route into each dynamic route folder

Your file tree should now look like this:

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
│       ├── playlist/
│       │   └── [playlistId]/
│       │       └── page.js
│       ├── search/
│       │   └── [searchId]/
│       │       └── page.js
│       └── video/
│           └── [videoId]/
│               └── page.js
├── .gitignore
├── eslint.config.mjs
├── jsconfig.json
├── next.config.mjs
├── package-lock.json
├── package.json
├── postcss.config.js
└── README.md
```

#### Now go to each route by typing `/route/[anything you want here]`

Also, notice **if you go to just `/route` there will be nothing now.** This is because page.js has been moved into the dynamic route, **so can no longer be found from /route.**

## Using params

The biggest reason why we use these dynamic routes is it makes it easy to pass information to the page via a URL.

In order to do so, we call the **params**.

- The *params* are the dyanmic route parameters, so whatever is passed in the URL can be called using *params*

```jsx
// Here is an example on how to call parameters


export default function Route({ params }) {
   const { id } = params;
   // IMPORTANT:
   // See how I call 'id', this should be replaced with what you titled
   // your folder, if you followed my naming convention you would put
   // searchId, videoId, or playlistId


   return (<p> This is a Route with the URL Containing {id} </p>);
}
```

### Implementing

Since all of our routes are dynamic, in every route, **call params and integrate it in someway to your component**, it does not have to make sense yet, that is for later.


> The code I gave you had an error, it is not a major error (for now), click **Fn + F12** and it will show you the error. Read the document it provides to correct your code
{: .prompt-danger }

**Answer below**

```jsx
// Just kidding, this is not the answer


// However, I will put the link to the documentation here, as it
// contains exactly what you need


// https://nextjs.org/docs/messages/sync-dynamic-apis


// As a programmer, always remember to use the documentation; it is there
// for a reason!
```
{: .nolineno }
{: .blur }

### Challenge Task

**Make the parameter in the /video/[ videoId ]/page.js determine the video id**

> HINT: Look at how the embedded video URL is formatted
{: .prompt-tip }

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

### Routing

Now that you know the difference between CSR and SSR, we will introduce another NextJS feature which can be used in CSR components.

When using **CSR Components**, we can now use `useRouter()`.

**What is useRouter()?**

- Basically NextJS's `<a>` tag for **CSR Components**
- Easy way to control navigation instead of relying on `<a>` tags
- Supports many different functions, shown below

```jsx
const router = useRouter();


// sends you to another route in your website
router.push('/specified/path');


// refreshes the current route you are at
route.refresh();


// Go back in your history
route.back();
```

## Implementing Client Side Rendering

We are going to have **2 components that use client side rendering**

- `app/video/[videoId]/page.js`
- `app/page.js`

**Why make these two Client Side?**

Web pages loading before everything is ready is fine, whereas showing incomplete results while in a search is not ideal.

### Challenge Task

> Make these two components client side components. When doing this, you will run into a couple errors. Attempt to Google or look at the documentartion for why you get these errors.
{: .prompt-warning }

The solution will be provided for this. Feel free to take a look if you get stuck or when you finish to ensure are working in the right direction.

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

**(remember to take a look at the example above)**

## Loading State

When you have Service Side Components, the entire webpage renders **on the server**. The main problem with this is: *If the content is not ready, what do we show the user?*

NextJS has a very simple and easy way to solve this issue; you can make a **static loading page**.

- If you have a SSR Component, in the **same folder**, make a `loading.js` file
- This `loading.js` file needs to be **static** so they can be ready right when someone loads the page.

This implementation is super simple, this is what your file tree should look like.

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

## Introduction to APIs and getting YouTube API key

### What is an API?

An API is an **Application Programming Interface**, which is a **set of rules that allow different software components to communicate with each other**.

Today, we will be working with **REST APIs** which is the most popular type of API. It communicates via HTTP methods which are GET and POST (there is also PUT and DELETE, but we are not going to use those).

- GET --> When you want to get data from the source.
- POST --> When you want to give data to the source.

### Example

Imagine you want to build a *Pokemon Information App*. The hard way to make this app is to collect every single bit of information about every single pokemon. **This is where an API could make the process much easier**. There is an API called `PokeAPI` where you send a request for Pokemon data, and it will send it back to you.

Here is how it works:
- You send an HTTP request (we will cover this in detail later)
- You specify what you want in the HTTP request. For example, you may want to know everything about Pikachu.
- The other end of the API will process this request, gather the information about Pikachu, then put it in a **JSON File** so you can understand it
- Lastly, they will send back the information you requested, and now you have all the information you needed without collecting any data yourself!

#### As a programmer, it would look like:

You send a request like this:

```jsx
const apiData = await fetch('https://pokeapi.co/api/v2/pokemon/pikachu')
```

You get something back that looks like this:

```jsx
// Note: This is an example, not what PokeAPI will actually send
{
 "name": "pikachu",
 "height": 4,
 "weight": 60,
 "types": [
   { "type": { "name": "electric" } }
 ]
}
```

After you get this information back, you can parse it and use it however you would like.

[Here is another example](https://www.youtube.com/watch?v=s7wmiS2mSXY&t=33s) if you are struggling a bit to understand.

## Enable the YouTube API

As previously mentioned, we are going to use the **YouTube API**. Luckily for us, it is free to use!

First thing we need to do is to **activate your Youtube API** and get your **API Key**.

Go to [Google Cloud Console](https://console.cloud.google.com/)

- If you have not made a project before, [follow this tutorial](https://youtu.be/dTT1RGW8eYw?feature=shared&t=17) to get one set up
- Next, on the home page, go to the **Navigation hamburger menu at the top left**
- Select `APIs & Services`, which is the 4th under **Products**. If you are stuck, go [here](https://console.cloud.google.com/apis/library)
- On the left, you will see a tab for `Library`. Click it
- In the search box, search for **"youtube data api v3"**. Click on the result, then click **Enable**
- You will be taken to the dashboard. In the middle-left of the screen, you will see three tabs:
    - **Metrics, Quota & System Limits, and Credentials**
- Select **Credentials**.
- On the right side of the screen, click `+ Create Credentials` and click `API Key`
- Follow any prompts (there should be none), then copy the **API Key** and save it somewhere safe for now. **DO NOT PUT THIS ANYWHERE ON THE INTERNET.**

**Congratulations!** You now have an API key for the Youtube API!

The reason why we need to get an API Key is to verify and authorize that we are allowed to call it. Most - if not all - APIs you will work with will need one.

Let's go back to your project. In the highest directory (the directory that holds src and .gitignore) **create a file and call it `.env`**.

In the `.env` file, paste your API Key like this:

```console
API_KEY=yourapikeyhere
```

Ensure you have a `.gitignore` file and add your `.env` to it. This way, the API key will not be posted to your Github repository if you want to upload your project there.

- NOTE: Even if you are not putting this on GitHub, you should still go through these safety measures. It is good practice to do so, as you don't want any of your frontend to accidentally show your API key.

## Creating your API

Now, it is time to make your **NextJS API**.

Making the API is about as easy as making the routes! In the `/app` folder, create a new folder titled `api`.

We need three different endpoints. (An API endpoint is a URL that acts as the point of contact between an API client and an API server):

- `app/api/playlist`
- `app/api/search`
- `app/api/video`

To create these endpoints, you need to add a `route.js` in each folder. **This is what your tree structure should look like now**:

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
│       ├── playlist/
│       │   └── [playlistId]/
│       │       ├── page.js
│       │       └── loading.js
│       └── api/
│           ├── playlist/
│           │   └── route.js
│           ├── search/
│           │   └── route.js
│           └── video/
│               └── route.js
├── .gitignore
├── eslint.config.mjs
├── jsconfig.json
├── next.config.mjs
├── package-lock.json
├── package.json
├── postcss.config.js
└── README.md
```

### Creating the API endpoints

We need to create our **GET** request for each route, as a reminder:

- A GET request is when you want to access data

NOTE: We will not POST, only GET, since we do not have a place to put data.

In every `route.js` put this here

```jsx
export async function GET(request) {
 // content goes here
}
```

You will notice that GET has the parameter `request`. That is where we find the details of the API request.

For example, when we call this API, this is what our request will look like:

- `/api/video?videoId=randomVideoId`

Notice this part: `?videoId=randomVideoId`

When you create an API request, this is how you will format it. After the question mark, put any parameters that the API accepts along with the input for it.

So for the one above, this API has the `videoId` parameter, and the input for it, comes after the equal sign.

Knowing this, we can add to the `api/video/route.js`

```jsx
export async function GET(request) {
 // Create a URL object from the incoming request
 const { searchParams } = new URL(request.url);
 // Extract the value of the "videoId" from the URL
 const videoId = searchParams.get("videoId");


 // make this example data for each endpoint for
 // now, this will be used to test the API
 const data = {'message':'Success'}


 // right now we do not have anything to really send back
 // but when we do, we send a response
 // a Response has a status (200 if successful) and a
 // a JSON object with the data
 return new Response(JSON.stringify(data), {
   status: 200,
   headers: { 'Content-Type': 'application/json' }
 });
}


```
{: file="app/video/page.js" }
{: .nolineno }

Now that you know how to create an endpoint, create the `route.js` for `/search` and `/playlist` given these two requests:

- `http://localhost:3000/api/search?text=lofi&type=video`
- `http://localhost:3000/api/playlist?playlistId=myplaylistid`

HINT: For `/search`, the `&` means there are MULTIPLE PARAMETERS, those being `text` and  `type`

### Test Your APIs

This part of your route

```jsx
const data = {'message' : 'Success'}
```

is there for a reason. Go to each route and type them into the URL of your browser that you are using. You should see this message (or whatever message you put) at the top left of the page.

If you do, congratulations! You have successfully made your first API! Otherwise, please go back and make sure everything looks the same.

## Calling YouTube API and Parsing the Data

Now that you've made our own API, we are now going to call the youtube API with our requests.

**Why did we make our own API to call the YouTube API?**

1. Creating our own API means we now have a backend, which parses all the data before sending the frontend the information to display. This keeps the frontend from knowing any extra data it doesn't need to know, adding a layer of abstraction and security.
2. It is better to learn how to make your own API and backend rather than just calling them.

### How to call the YouTube API

The url for the API is: `https://www.googleapis.com/youtube/v3`.

Everything you need to know about the YouTube API is [here in the documentation](https://developers.google.com/youtube/v3/docs).

Additionally, I've provided an example of calling the YouTube API in `/video`. As you can see, there is no need to **manually type the URL**; we have tools to make it easier and guarantees it to be correct:

```jsx
 // we create a new URL object, and put the base URL we want to call
 const url = new URL("https://www.googleapis.com/youtube/v3/videos");


 // Next for the API parameters
 // instead of straight the parameters into the URL, we can
 // just set them using our URL object
 url.searchParams.set("part", "snippet,contentDetails,statistics");
 url.searchParams.set("id", videoId);


 // Remember the API key we added to our env file?
 // Well javascript lets us call it using process.env.API_KEY
 // ANYTIME WE CALL THE API WE NEED THE API KEY FOR AUTHORIZATION
 url.searchParams.set("key", process.env.API_KEY);
```

Feel free to add this to your `/app/api/video/route.js`.

Note that APIs will not always work. Sometimes, when you call an API, it may return a unsuccessful code, like **500**. We need to make sure to account for these situations.

Thankfully, we can use a `try and catch` statement.

Here is the full `/app/api/video/route.js`:

```jsx
export async function GET(request) {
   // get the parameters from the request
   const { searchParams } = new URL(request.url);
   const videoId = searchParams.get("videoId");

   // Build the url to call the API
   const url = new URL("https://www.googleapis.com/youtube/v3/videos");
   url.searchParams.set("part", "snippet,contentDetails,statistics");
   url.searchParams.set("id", videoId);
   url.searchParams.set("key", process.env.API_KEY);

   // make the attempt to call the API
   try {
       const response = fetch(url.toString());

       // we can check if it went okay, if now we want to raise
       // an error to show the API access was unsuccessful
       if (!response.ok) {
           const errorText = await response.text();
           console.error("YouTube API Error:", errorText);
           throw new Error("Failed to fetch video details");
       }

       // If it is fine, that means we can take the JSON
       // string and convert to a JSON Object so we can parse
       // the data
       const data = response.json();

       // since this was successful, we return a Response
       // with the status 200, and a json string of the data
       return new Response(JSON.stringify(data), {
           status: 200,
           headers: { 'Content-Type': 'application/json' }
       });
   } catch (error) {
       // If the API ends up not working, return why it did not
       // work, and make the status 500
       return new Response(JSON.stringify({ error: error.message }), {
           status: 500,
           headers: { 'Content-Type': 'application/json' }
       });
   }
}
```

> THERE WILL BE A COUPLE ERRORS IN THIS FILE Use the console and NextJS helper to figure out these errors. There may be a couple things you need to add for security that were not included.
{: .prompt-danger }

### Challenge Task

Now it is your turn! Use the [YouTube API Documentation](https://developers.google.com/youtube/v3/docs) and any other resources **(TRY TO AVOID USING AI)** to make the API calls for `/api/search` and `/api/playlist`.

Everything you need to complete this task has already been covered. I will provide the solution - with a couple errors - for `/api/search` without any edge-case checking, but not for `/api/playlist` since is very similar.

### Partial Solution to Challenge Task

```jsx
export function GET(request) {
   const { searchParams } = new URL(request.url);
  
   const text = searchParams.get('text');
   const channel = searchParams.get('channel');
   const type = searchParams.get('type');

   const url = new URL("https://www.googleapis.com/youtube/v3/search");

   url.searchParams.set("part", "snippet");
   url.searchParams.set("q", text);
   url.searchParams.set("type", type);
  
   if (type === "video") {
       url.searchParams.set("videoDuration", "medium");
   }

   const response = await fetch(url.toString());
  
   if (!response.ok) {
       console.log(response);
       throw new Error('Network response was not ok');
   }

   const data = response.json();

   return new Response(JSON.stringify(data), {
       status: 200,
       headers: { 'Content-Type': 'application/json' }
   });
}
```
{: file="app/api/search/route.js" }
{: .nolineno }
{: .blur }

## Calling Custom API to the Front End to Display the Data

Now that we actually have data to put in our website, we can now display everything.

First, we call the API in `/app/search`:

```jsx
 // encodeURIComponent is necessary since the searchId may contain
 // weird non unicode characters, this will fix that.
 const res = await fetch(
     `http://localhost:3000/api/search?text=${encodeURIComponent(searchId)}&type=video`,
     { cache: "no-store" } // this is optional
                           // included so there is always
                           // new / fresh data
 );
```

Just like how we called the YouTube API, we can call our own API. When we receieve the data, we can display it.

### How to Display API Data

> **Try this on your own**. Figure out what the API Data looks like and how you can display it. I will give a full solution to this one and a partial solution to another. You will need to do that last one on your own.
{: .prompt-warning }


Here is what an example video search that returns two videos would look like:

```json
{
 "kind": "youtube#searchListResponse",
 "etag": "dummyEtag123",
 "regionCode": "US",
 "pageInfo": {
   "totalResults": 2,
   "resultsPerPage": 2
 },
 "items": [
   {
     "kind": "youtube#searchResult",
     "etag": "etag1",
     "id": {
       "kind": "youtube#video",
       "videoId": "dQw4w9WgXcQ"
     },
     "snippet": {
       "publishedAt": "2023-01-01T00:00:00Z",
       "channelId": "UC123456789",
       "title": "Relaxing Lofi Beats",
       "description": "Perfect background music for studying and relaxing.",
       "thumbnails": {
         "default": {
           "url": "https://i.ytimg.com/vi/dQw4w9WgXcQ/default.jpg"
         }
       },
       "channelTitle": "Lofi Radio",
       "liveBroadcastContent": "none"
     }
   },
   {
     "kind": "youtube#searchResult",
     "etag": "etag2",
     "id": {
       "kind": "youtube#video",
       "videoId": "hY7m5jjJ9mM"
     },
     "snippet": {
       "publishedAt": "2023-01-02T00:00:00Z",
       "channelId": "UC987654321",
       "title": "Chillhop Essentials - Winter 2023",
       "description": "A selection of jazzy beats to relax or code to.",
       "thumbnails": {
         "default": {
           "url": "https://i.ytimg.com/vi/hY7m5jjJ9mM/default.jpg"
         }
       },
       "channelTitle": "Chillhop Music",
       "liveBroadcastContent": "none"
     }
   }
 ]
}
```

> If the above JSON is unreadable and confusing to you, do not worry. Paste any JSON object into [this website](https://jsoncrack.com/editor); it will help you visualize the data better.**
{: .prompt-tip }

Now that you know what the API data may look like, you can get to the data you need. For example:

- **Thumbnail** = `items[i].snippet.thumbnails.medium.url`
- **Title** = `items[i].snippet.title`
- **Description** = `items[i].snippet.description`
- **Video ID** = `items[i].id.videoId`

> **NOTE:** I expect you to know how to get info from JSON objects, if not, here is a [quick demonstration](https://www.youtube.com/watch?v=iiADhChRriM).
{: .prompt-warning }


To show all the videos, we need to **iterate through all the videos in the items array and display a video card for each one**.

This is easily done with JavaScript's map function:

```jsx
const videos = await res.json();


return (<div className="flex flex-col items-center w-full overflow-y-auto pb-10">
 {videos.items?.map((vid) => { // go through every video in the items

    // cant use useRouter() since this is SSR
     return (
      <a href={`/video/${vid.id.videoId}`}>
         <img
             src={vid.snippet.thumbnails.medium.url}
             alt={vid.snippet.title}
             width={160}
             height={120}
         />
         <div>
             <h2> {vid.snippet.title} </h2>
             <p> {vid.snippet.description} </p>
         </div>
     </a>);
   });
 }
</div>);
```

Your full `/video/page.js` should look like this:

```jsx
"use client";

import { useParams } from "next/navigation";
import { useEffect, useMemo, useState } from "react";


export default function Content() {

 const { videoId } = useParams();

 // since this will be rendered later, since embed is an API
 // we can use useState() to set the video
 const [video, setVideo] = useState(null);

 // fetch the video and set it to the video variable
 // if there is one
 // review useEffect and useState if this seems confusing
 useEffect(() => {
   const fetchVideo = async () => {
     const res = await fetch(`/api/video?videoId=${encodeURIComponent(videoId)}`, { cache: "force-cache" });
     const vid = await res.json();
     setVideo(vid?.items?.[0] ?? null);
   };

   fetchVideo();
 }, [videoId]);

 // get the snippet from the video to extract
 const title = video?.snippet?.title ?? "No title";
 const description = video?.snippet?.description ?? "No description";

 // now display all the information
 return (
   <>
     <div className="h-lvh w-lvw flex flex-col items-center justify-start text-white">
       <h1 className="text-3xl p-2 m-2">{title}</h1>
       <iframe
         width="960"
         height="540"
         src={`https://www.youtube.com/embed/${videoId}`}
         title={title}
         frameBorder="0"
         allowFullScreen
       ></iframe>
       <div className="flex items-center justify-between max-h-1/4 w-full max-w-5xl p-4 mt-4 bg-neutral-700 rounded-lg overflow-scroll"> 
        <p className="mt-4 text-gray-300">{description}</p>
       </div>
     </div>
   </>
 );
}
```
{: file="app/video/[videoId]/page.js" }
{: .nolineno }
{: .blur }

Please give an attempt at making the playlist one. You will need to:

- Call your playlist API to retreive the playlist
- Go through the items (videos) in your playlist and display them as cards
- Make sure each video links to the `/video/[videoId]/page.js` of that video

> A partial solution with errors is given, but given the demonstrations provided above, you should have more than enough to complete this on your own.
{: .prompt-warning }

**Solution (with errors)**

```jsx
export default function PlaylistPage({ params }) {
  
   const { playlistId } =  params;

   const res = fetch(
       `http://localhost:3000/api/playlist?playlistId=playlistId`,
       { cache: 'no-store' }
   );

   const videos = res.json();

   return (
       <>
       <div className="w-screen h-screen flex flex-col items-center justify-start text-white">
           <h1 className="text-4xl mt-6 mb-4">Playlist Videos</h1>
           <div className="flex flex-col items-center w-full overflow-y-auto pb-10">
           {video.items.map((p) => {
               return (
               <a href={`/video/${p.snippet.resourceId.videoId}`} key={p.snippet.resourceId.videoId} className="w-1/2 max-w-3xl bg-neutral-700 rounded-2xl p-3 m-3 flex items-start gap-3 max-h-1/5">
                <img
                    src={p.snippet.thumbnails.medium?.url}
                    alt={p.snippet.title}
                    width={160}
                    height={120}
                    className="rounded-md shrink-0 object-fill"
                    style={{ width: '160px', height: '120px' }}
                />
                <div className="xflex flex-col justify-start h-full p-3">
                    <h2 className="text-lg font-semibold leading-tight mb-1"> {p.snippet.title} </h2>
                    <p className="text-sm text-gray-300 leading-snug max-h-3/4 overflow-scroll"> {p.snippet.description} </p>
                </div>
               </a>
           );
           })}
           </div>
       </div></>
   );
}
```
{: file="app/playlist/[playlistId]/page.js" }
{: .nolineno }
{: .blur }

## Final Challenge Task

Lastly, we need our `/search/[searchId]/page.js`:

- Call your playlist API to retreieve the playlist
- Go through the items (videos) in the search and display them as cards
- Be aware of edge cases! Find a way to not display shorts or any channels. (Yes, even if you specify "videos", they sometimes show up)
- Each card should link to the `/video/[videoId]/page.js` of that video

Your solution will be very similar to the playlist page. Utilize anything in this tutorial, Google, and documentation to help you.

## Where to go from here?

### Congrats!

You have finished the main part of this tutorial!

There is a ton more you can do with this project, but you should have all the resources you need now to take it from here.

Here is a limited list of things you may consider for improving your website:
- Utilize the button we did not program yet (search for playlist)
- Edit "search" to accomodate playlists, or you can make a seperate playlist search (second would be easier, first is more ideal in OOD)
- Make a way to go through playlist videos
- Make a channel page
- Make a channel search page
- Show comments
- Show related videos
- An ambitious one could be to connect to your youtube account / use OAuth to view your own private content and playlists.
- Improving the design and overall look of the website

Overall, you should do whatever you want to make this project your own. Be creative and take it where you want it to go. If you thought of a different way to improve the website, do it! If you want to add something you did not see in this tutorial, don't be afraid to do it; use your resources and make it happen!

Thank you for sticking to the end. I hope you enjoyed the ride and learned something new!