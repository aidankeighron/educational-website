---
title: "OPTIONAL: TailwindCSS"
parent_post: FocusTube
module_number: 4
layout: module
media_subpath: /assets/tutorials/focustube
---

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
