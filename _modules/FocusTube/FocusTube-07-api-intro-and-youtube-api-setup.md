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

> New to APIs? Check out our [What is an API?]({{ '/references/what-is-an-api/' | relative_url }}) background page before continuing.
{: .prompt-info }

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
