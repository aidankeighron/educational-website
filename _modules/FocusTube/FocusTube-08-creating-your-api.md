---
title: "Creating your API"
parent_post: FocusTube
module_number: 8
layout: module
media_subpath: /assets/tutorials/focustube
---

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
