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
