---
title: "Dynamic Routing and Using Params"
parent_post: FocusTube
module_number: 5
layout: module
media_subpath: /assets/tutorials/focustube
---

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
