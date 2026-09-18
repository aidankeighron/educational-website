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
7. Select Public data, we aren't accessing youtube's private user data
7. Copy the generated **API Key** and save it somewhere safe for now. **DO NOT POST THIS ANYWHERE ON THE INTERNET.**

**Congratulations!** You now have an API key for the YouTube API.

### Securing Your API Key

Because your API key acts as a password to your Google account's API quota, you must hide it from the public.

1. In the root directory of your project (the same folder that holds `package.json`), create a new file named `.env`.
2. Open the `.env` file and paste your API Key like this:

```console
API_KEY=yourapikeyhere
```

3. Ensure you have a `.gitignore` file and that `.env` is listed inside it. This guarantees that your API key will never be uploaded to GitHub.

> **NOTE:** Even if you aren't using GitHub right now, you should still practice these safety measures. For a deeper dive into why this matters, check out our [Environment Variables & Secret Safety]({{ '/references/environment-variables/' | relative_url }}) background page.
{: .prompt-info }
