---
title: Syllabus To CSV
author: aidan
date: 2025-03-23 12:00:00 +0800
categories: [JavaScript, Syllabus To CSV]
tags: [JavaScript, Easy]
description: Convert your class syllabus into a CSV of all your assignments
comments: false
pin: true
media_subpath: /assets/tutorials/syllabus to csv
---


## About the project

Welcome to the Syllabus To CSV tutorial! In this project, you will learn the fundamentals of JS and learn how to create a project from start to finish.

**What you will learn:**

- How to setup and run a chrome extension.
- How to call REST APIs and process their responses.
- Create a basic UI with HTML and CSS
- Interact with DOM elements with JS

**What you will make:**

By the end of this tutorial, you will have a working chrome extension that takes a class syllabus and converts it into a CSV with all your class assignments.

**Further possibilities:**

Once you've completed this tutorial, you can expand on it in many ways! You could add different types of conversions, convert the current webpage to CSV or even an image. You can also use it to extract different information out of the syllabus, grade charts or instructor list. The possibilities are endless.

## Setup

We are going to create a new GitHub repo for this project, no starter code needed.

- Navigate to GitHub desktop
- Click on the repository dropdown
- Select `Add` > `Create new repository`
- Give it a Name and *optionally* a description
- Click create repository
- Open in your preferred IDE

Create a file called `manifest.json`, make sure to give it a name and description.

```json
{
    "name": "YOUR NAME HERE",
    "description": "YOUR DESCRIPTION HERE",
    "version": "1.0",
    "manifest_version": 3,
    "action": {
      "default_popup": "index.html",
    },
}
```
{: file="manifest.json" }

This is a setup file that tells your browser some basic info about your extension. You are telling it to use `index.html` as the popup, the name, description, and version of your extension along with the version of the manifest you want to use (the different manifest versions just change what parameters you have access too). There are a lot of different things you can add to the manifest.

- Images for your extension
- Background scripts
- Permissions
- Much more

Next lets create the base popup HTML. Create a file called `index.html`, give it a title (this can be the same as the manifest name)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>EXTENSION NAME</title>
</head>
<body>
    <h1>My super cool extension</h1>
</body>
</html>
```
{: file="index.html" }

This is telling the browser what you want the popup to look like. In the `head` tag you are specifying the metadata

- What character set use
- How big the popup should be (default width)
- What the title of the popup should be (this is more useful with websites but still important to set)

Next in the `body` tag we are defining the content of the extension. For now we are just adding some text.

### Set up your Browser

We will be using chrome as an example but it should be similar for most other browsers.

- Navigate to your extensions page [chrome://extensions/](chrome://extensions/){:target="\_blank"}
- Make sure to toggle it into developer mode (top right corner)
- Select `Load unpacked` and select the root folder (outermost folder) of your GitHub repo
- You should see your extension in the extension list

## Running the project

You should see your extension in your extension list (you might need to pin it). All you need to do to "run" your extension is to click on its icon. For debugging you can right-click on the icon and select `Inspect popup` to open dev tools. Whenever you make a change make sure to go to the extensions page and click the reload icon at the bottom of your extensions card.

> Make sure you reload your extension after changing anything or you will not see the changes.
{: .prompt-info }

## Create a basic UI

Let’s improve the popup so users can upload their syllabus file for us to use!

Within our `<body>` element, we’ll add an `<input>` element with the type set to `"file"` — this allows users to select and submit their syllabus and add a respective id which we will use for our event listeners. Just below the input, we’ll include a `<p>` tag to let users know where they’ll be able to click and download their CSV file, which can contain text like **Click here to download your file!**.

> ✅ **Tip:** Make sure to properly close both the `<input>` and `<p>` tags.

Next, we need to connect our JavaScript to this HTML. To do that, we’ll add a `<script>` tag right before the closing `</body>` tag. The script should have `type="module"` and `src="popup.js"`.

Using `type="module"` allows us to use modern JavaScript features such as `import` and `export` statements. The `src` attribute tells the browser to load the logic from our `popup.js` file — the place where all our “behind-the-scenes” functionality will live. Later, we’ll also update the `innerHTML` of the `<p>` tag to contain a downloadable link once the file has been processed.

> Make sure to include the script tag at the bottom of the body or else your JavaScript will not work on your HTML elements.
{: .prompt-info }

## Handle file uploads

Now that we’ve set up a file input, let’s write the JavaScript needed to handle uploaded files and prepare them for processing.

We’ll be using `document.getElementById()` to grab our file input and attach an event listener that runs every time a user selects a file.

```js
document.getElementById('file-upload').addEventListener('change', async () => {
    // all the lines below will go inside here
});
```
Here, we’re listening for the "change" event on the file input id (or whatever id you used) — this fires whenever a user picks a file.
```js
async () => {}
```
Async allows us to use await inside the function. Since we'll likely send the file to an external API (which takes time), we want to pause and wait for the response before moving on — this keeps the code clean and readable.

Without async, we’d have to use .then() chains, which are harder to manage.

Arrow function syntax (() => {}) is a modern way to write functions in JavaScript. It's short, clean, and avoids creating its own this context — which works well here since we don’t need to refer to the event handler’s context directly.

> ✅ **Tip:** In short we use async () => {} to write cleaner, more modern code that lets us easily work with APIs that take time to respond.

```js
const fileUploaded = this.files.item(0);
```

This line grabs the first file the user selected. Since we’re only supporting one file at a time, we access the file at index 0.

Below you should create your own safety check to check if fileUploaded is null. If it is, we want to return.

```js
const form = new FormData();
form.append('purpose', 'ocr');
form.append('file', new File([fileUploaded], `${fileUploaded.name}`));
```
Here, we create a FormData object to prepare for sending the file to an external API.

FormData works like a key-value map that you can send with fetch() for things like file uploads.

We add two things:

A purpose field — this is useful if your API requires it (in this case, to label it for OCR processing).

The actual uploaded file, wrapped in a new File object.

> ✅ **Note:**  Wrapping the file again with new File([...]) is optional but helpful if you want to manipulate the name or metadata before sending.

> Important: All of these lines (fileUploaded, if (fileUploaded == null), and the FormData block) should be written inside the event listener function — directly with the async () => {} function.
{: .prompt-info }

This is the foundation of getting the syllabus file from the user and preparing it for conversion.

In the next step, we’ll send this FormData to an OCR model for parsing and response.

## Getting Your first API Key

Before we can send our syllabus to an AI model, we need an API key to authenticate with [Mistral OCR](https://mistral.ai/news/mistral-ocr), the model we’ll be using to process and extract text from PDF files.

### What is Mistral OCR?

Mistral OCR is a powerful AI model that can extract structured information from scanned documents, including PDFs — which is exactly what we need for turning a syllabus into a list of assignments.

> 🔗 Learn more: [Mistral OCR announcement](https://mistral.ai/news/mistral-ocr)

---
### 🌐 What is an API?

Before we use Mistral OCR, let’s take a quick step back and understand **what an API actually is**.

An **API (Application Programming Interface)** is a way for two programs to talk to each other. In our case, we’ll be using JavaScript to talk to an external AI service (Mistral) — and that conversation happens through an API.

> 🧠 Think of it like placing an order at a restaurant: you (the client) tell the waiter (the API) what you want, and the waiter brings it from the kitchen (the server). You don’t need to know how the kitchen works — just how to place an order properly.

### 🎥 Helpful Videos

- ▶️ [**What is an API?** (by Simply Explained)](https://www.youtube.com/watch?v=ByGJQzlzxQg&t=9s)  
  *This video explains APIs using real-world analogies — perfect if you're just starting out.*

- ▶️ [**4 Most Important HTTP Requests That Can Be Made to an API**](https://www.youtube.com/watch?v=tkfVQK6UxDI)  
  *This breaks down the core HTTP methods you'll use when working with APIs: GET, POST, PUT, and DELETE.*

---

### What is an API Key?

An API key is like a password that allows your project to communicate with a third-party service (in this case, Mistral). It tells the API who you are and whether you’re allowed to use it.

Think of it like a secret access badge — you’ll need one to send your file and get a response from Mistral.

---

### Step 1: Get your Mistral API Key

1. Go to [https://console.mistral.ai/api-keys](https://console.mistral.ai/api-keys)
2. Log in or create an account
3. Click **“Create API Key”**
4. Copy the key — it will look something like:  mistral-key-abc1234567890
   
---

### Step 2: Create a `hidden.js` file

To keep your API key separate from your main code (and avoid accidentally uploading it), let’s store it in a new file.

Create a file called: hidden.js

And inside it, write:

```js
const mistralApiKey = "your-mistral-api-key-here";
const geminiApiKey = "your-gemini-api-key-here"; // for using Gemini later

export default {
  mistralApiKey,
  geminiApiKey
};
```
> Never commit this file to GitHub!
If you’re using Git, be sure to add hidden.js to your .gitignore.
{: .prompt-danger }

### Step 3: Import your API keys
In your popup.js, import them like this:
```js
import apiKeys from "./hidden.js";
const mistralApiKey = apiKeys.mistralApiKey;
const geminiApiKey = apiKeys.geminiApiKey;
```

You're now ready to securely connect to Mistral and begin sending files for parsing! Next up: we’ll write the code that sends our FormData to the Mistral OCR API!

## Convert upload to PDF

Now that we’ve built our `FormData` object containing the uploaded syllabus file, we’re ready to send it to **Mistral’s OCR API** for processing.

We’ll do this using JavaScript’s `fetch()` function — this allows us to make requests to APIs directly from the browser.

Here’s the full function:

```js
/**
 * Convert a PDF to a JSON object
 * 
 * @param {FormData} form 
 * @returns {Promise<Object>}
 */
async function PDFToJson(form) {
    const uploadedPDF = await fetch('https://api.mistral.ai/v1/files', {
        method: 'POST',
        headers: {
            "Authorization": `Bearer ${mistralApiKey}`
        },
        body: form,
    });

    const PDFJson = await uploadedPDF.json(); 

}
```
🧠 Let’s break it down step-by-step:
### The comment block at the top
```js
/**
 * Convert a PDF to a JSON object
 * 
 * @param {FormData} form 
 * @returns {Promise<Object>}
 */
```
This is a JSDoc-style comment, which is a great practice even in beginner projects. It tells other developers (or future you):

What this function does

What kind of argument it expects (FormData)

What it returns (a promise that resolves to a JSON object)

> 💡 Writing clear comments like this helps others understand your code quickly and makes your project easier to maintain or expand in the future.
{: .prompt-info }

###  Fetch
```js
fetch('https://api.mistral.ai/v1/files', { ... })
```
This is the URL of Mistral’s file upload API. When we call this, we’re telling Mistral:

> “Hey, I want to upload a file for OCR processing.”
```js
method: 'POST'
```
This tells the API we want to send data (in this case, the file).
There are other methods like GET, PUT, and DELETE, but POST is most common for sending form or file data.

### Headers
```js
headers: { "Authorization": "Bearer ... " }
```
APIs often require authentication — this is how they know who you are.

The "Authorization" header tells the API,

"Here’s my API key — please allow me to use your service."

"Bearer" is the standard keyword used to pass tokens securely.

You should already have your API key stored in hidden.js, and here we’re inserting it using backticks and ${} for string interpolation.

### Body
```js
body:form
```
This is the actual file upload!
We’re sending the FormData object we created earlier (which includes the PDF file) as the body of the request.
```js
await uploadedPDF.json()
```
Once Mistral finishes processing the file, it sends back a response — usually in JSON format.
> If you **don’t** call `.json()` and just look at the `response` object itself, you’ll get a **network response object**, not the actual data.
We call .json() on the response to convert it into an object we can work with in JavaScript.

### What does Mistral send back?
It doesn’t send back the converted syllabus — not yet.

Instead, it responds with file metadata, like this:

```json
Copy
Edit
{
  "id": "file-abc123",
  "status": "uploaded",
  "filename": "syllabus.pdf",
  "created_at": "2024-03-23T15:12:00Z"
}
```
This response tells us:

The upload was successful ✅

We now have a file ID that we can use to request a signed download URL in the next step

The OCR processing hasn’t happened yet — we’ll request it next!

> Important: This function does not do OCR yet. It only uploads the file and gives back an ID.
We'll use this ID in a follow-up request to get the downloadable link and send that to Mistral's OCR model.
{: .prompt-info }

### Upload PDF to get URL

Now that we’ve uploaded the file, Mistral gave us a **file ID** in the response. We’re going to use that ID to request a **signed file URL** — a secure, temporary link to download or reference the uploaded file.

---

### 🚧 Your Turn: Make the API Call

Use the `fetch()` function to make a **GET request** to this endpoint: https://api.mistral.ai/v1/files/FILE_ID/url?expiry=24
> Replace `FILE_ID` with the ID you received from the previous step (`PDFJson.id`)

This tells Mistral:  
> “Please give me a temporary link to access the file I just uploaded.”

The `expiry=24` part means the link will only work for **24 hours**.

---

### 🔐 Headers You’ll Need

Your request should include a `headers` object with the following:

```js
headers: {
  "Accept": "application/json",
  "Authorization": `Bearer ${mistralApiKey}`
}
```

What does "Accept": "application/json" mean?
This tells the server:

“Hey, I expect the response to be in JSON format.”

Without it, some APIs may return unexpected formats or not work as intended.

### ✅ Your Goal
Make the fetch() call using the correct method (GET)

Pass in the required headers

Use .json() to extract the result into a usable object (just like we did when uploading the file)

Once you’ve done that, you’ll have access to a temporary URL like:
```json
{
  "url": "https://cdn.mistral.ai/files/abc123/syllabus.pdf?token=..."
}
```
We’ll use that URL in the next step when we send the file to Mistral’s OCR model for analysis!

> Hint: Store the result in a variable like responseJSON, then access the URL with responseJSON.url

### Parse PDF to Markdown

## Parse upload into assignment list

### Sending upload to gemini

### Downloading the file

## Extending your extension
