---
title: "API Intro and YouTube API Setup"
parent_post: React-FocusTube
module_number: 7
layout: module
media_subpath: /assets/tutorials/focustube
---

## What You Will Learn in This Section

- How API endpoints, requests, and responses work in a client-only mobile application.
- How to structure an API helper layer in React Native for cleaner, more secure code.
- How to connect, fetch, parse, and display live YouTube data in your mobile UI using React hooks.

> New to APIs? Check out our [What is an API?]({{ '/references/what-is-an-api/' | relative_url }}) background page before continuing — it covers what an API is, REST/HTTP methods, and what an API key is.
{: .prompt-info }

## Transitioning to the YouTube API

Just like PokeAPI provides us with Pokémon data, Google provides a **YouTube API** that lets developers access real YouTube videos, playlists, and channel data programmatically. 

Instead of showing Pikachu's stats, we will be requesting real video search results to power FocusTube!

### Enabling the YouTube API

Unlike PokeAPI, Google needs to know who is requesting their data to prevent abuse. To do this, we need to generate a unique "password" called an **API Key**.

Go to the [Google Cloud Console](https://console.cloud.google.com/) to get started:

1. If you have not made a project before, [follow this quick tutorial](https://youtu.be/dTT1RGW8eYw?feature=shared&t=17) to get one set up.
2. Next, open the **Navigation hamburger menu** at the top left.
3. Select **APIs & Services** > **Library**.
4. In the search box, search for **"youtube data api v3"**. Click on the result, then click **Enable**.
5. You will be taken to the dashboard. In the middle-left of the screen, select the **Credentials** tab.
6. On the right side of the screen, click **+ Create Credentials** and select **API Key**.
7. Select Public data, we aren't accessing youtube's private user data.
8. Copy the generated **API Key** and save it somewhere safe for now. **DO NOT POST THIS ANYWHERE ON THE INTERNET.**

**Congratulations!** You now have an API key for the YouTube API.

### Securing Your API Key in React Native

Because your API key acts as a password to your Google account's API quota, you must hide it from the public.

In Web frameworks like NextJS, we can safely hide our API keys in a backend server that the client cannot see. However, React Native compiles into a mobile app that runs **entirely** on the user's phone. 

> **WARNING:** In production-ready mobile apps, client-side API keys can be extracted by hackers reverse-engineering your app's code. To secure it fully, you would build a separate backend server to act as a proxy. For this educational project, we will use Expo's built-in client-side environment variable system.
{: .prompt-warning }

Let's set up environment variables in Expo:

1. In the root directory of your project (the same folder that holds `package.json`), create a new file named `.env`.
2. Open the `.env` file and paste your API Key like this:

```console
EXPO_PUBLIC_API_KEY=yourapikeyhere
```

> **IMPORTANT:** In Expo, all environment variables must start with the prefix `EXPO_PUBLIC_` for the app to access them!
{: .prompt-danger }

> **QUESTION:** Why must all Expo environment variables start with the prefix `EXPO_PUBLIC_`? What would happen if we used private API keys without this prefix on a client-side device?
{: .prompt-tip }

3. Ensure you have a `.gitignore` file and that `.env` is listed inside it. This guarantees that your API key will never be uploaded to GitHub.

> For a deeper dive into environment variables, why leaked keys are dangerous, and how `EXPO_PUBLIC_` variables are shipped inside your app bundle, see our [Environment Variables & Secret Safety]({{ '/references/environment-variables/' | relative_url }}) background page.
{: .prompt-info }
