---
title: "File-Based Routing and Adding Routes"
parent_post: FocusTube
module_number: 2
layout: module
media_subpath: /assets/tutorials/focustube
---

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
