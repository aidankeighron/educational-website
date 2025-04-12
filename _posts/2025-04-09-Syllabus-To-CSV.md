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

> Important: All of these lines (fileUploaded, if (fileUploaded == null), and the FormData block) should be written inside the event listener function — directly under the async () => {} line.
{: .prompt-info }

This is the foundation of getting the syllabus file from the user and preparing it for conversion.

In the next step, we’ll send this FormData to an OCR model for parsing and response.

## Convert upload to PDF

### Upload PDF

### Parse into JSON

## Parse upload into assignment list

### Sending upload to gemini

### Downloading the file

## Extending your extension
