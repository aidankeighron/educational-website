---
title: "File-Based Routing and Adding Routes"
parent_post: FocusTube
module_number: 2
layout: module
media_subpath: /assets/tutorials/focustube
---

## File-Based Routing

**File-Based Routing** means that the structure of your files and folders inside the **app/** directory *automatically* defines the routes in your web app.

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

> **QUESTION:** What is a component? They are key to truly mastering the power of react, so learn a high level definition to keep in the back of your mind since this term will come up again and again.
{: .prompt-tip }

For more information, please feel free to use the [NextJS Documentation](https://nextjs.org/docs/app/building-your-application/routing/route-handlers).

<!-- > **QUESTION:** What is a 'route'? Give an example of when you might need one
{: .prompt-tip } -->

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

> **BUG:** In the above answer I purposely **mistyped** one of the attributes of `<iframe>`.
{: .prompt-danger }

Luckily, when you save the file and look at it at the ```/video``` web app route, you will see **a NextJS logo that says issue on the bottom left**.

If you click this logo, it will tell you any errors or warnings you may currently have in your file. This will be a massive help while you are programming. However, it won't catch everything, such as *bugs*! Bugs may not be an error; they just don't do what you intended them to do.

When this happens, make sure you always use **Fn+F12**, which lets you inspect your webpage HTML and CSS. Additionally, if you are stuck or unsure what is going on, you can always use various console functions like ```console.log()```, which is the equivalent to ```print()``` in javascript. **I will add areas for you to try these out later**.

### Search Route

Next up, we have our `/search` route.

> **NOTE:** This may sound counter-intuitive, but the search page will display the search results, not the actual search bar. The search bar will be added on the *home page*, which we will get to shortly.
{: .prompt-info }

First, let's create a basic page component that just returns a title.

```jsx
export default function SearchPage() {
 return (
   <div>
     <h1>Search Results</h1>
     {/* Video cards will go here */}
   </div>
 );
}
```
{: file="app/search/page.js" }
{: .nolineno }

> **QUESTION:** This is the second component we have seen. You may start to pickup a pattern on how they are constructed. What do these component functions return?
{: .prompt-tip }

Next, we need a way to mock a single "Video Card". A video card should have a thumbnail image, a title, and a description. We also want the entire card to be clickable so it can take the user to the `/video` route we just made! A perfect job for a `<Link>` component.

Let's import `<Link>` and add a dummy video card below our title:

```jsx
import Link from 'next/link';

export default function SearchPage() {
 return (
   <div>
     <h1>Search Results</h1>
     <div>
       <Link href="/video">
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
       </Link>
     </div>
   </div>
 );
}
```
{: file="app/search/page.js" }
{: .nolineno }

Wait, what is a `<Link>` component? We defined a 'component' above. It is just that. Just prebuilt for you by React. Using this prebuilt component gives us another really cool ability...

> **QUESTION:**  Why do we import the `<Link>` component instead of just using a standard `<a>` tag? What happens in the browser if you use a standard `<a>` tag?
{: .prompt-tip }

You can copy and paste the `<Link>` block multiple times if you want to see what a list of results looks like. Later in the tutorial, we will replace this hardcoded fake data with real API calls!

### Playlist Route

> **TASK:** Make the playlist route. It is very similar to the search route. You got this!
{: .prompt-warning }

- Inside ```/playlist/page.js```, create a React component that shows fake video cards/items in a playlist
- Style each video as you want
- Make sure each one links to the ```/video``` route
