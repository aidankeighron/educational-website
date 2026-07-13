---
title: "API Intro and YouTube API Setup"
parent_post: FocusTube
module_number: 7
layout: module
media_subpath: /assets/tutorials/focustube
---

## What You Will Learn in This Section

- How to use and create the fundamental parts of an API (endpoints, requests and responses) in NextJS
- How to connect, fetch, parse, and display API data from YouTube

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
