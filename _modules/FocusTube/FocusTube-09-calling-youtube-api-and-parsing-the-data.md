---
title: "Calling YouTube API and Parsing the Data"
parent_post: FocusTube
module_number: 9
layout: module
media_subpath: /assets/tutorials/focustube
---

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
> **QUESTION:** Looking at the search parameters in the code above (`part`, `q`, `type`, `videoDuration`), what do you think each of these represents when we're requesting data from the YouTube API?
{: .prompt-tip }

> **TASK: Complete the Playlist API Endpoint**
> Use the partial solution above as a guide to create the `/api/playlist/route.js` endpoint! You will need to check the YouTube API documentation to see what specific parameters the Playlist endpoint requires.
{: .prompt-warning }
