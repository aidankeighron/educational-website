---
title: "Basic UI and File Uploads"
parent_post: Syllabus-To-CSV
module_number: 2
layout: module
media_subpath: /assets/tutorials/csv
---

## Create a basic UI

Let’s improve the popup so users can upload their syllabus file for us to use!

Within our `<body>` element, we’ll add an `<input>` element with the type set to `"file"` — this allows users to select and submit their syllabus and add a respective id which we will use for our event listeners. Just below the input, we’ll include a `<p>` tag to let users know where they’ll be able to click and download their CSV file, which can contain text like **Click here to download your file!**.

> **Tip:** Make sure to properly close both the `<input>` and `<p>` tags.
{: .prompt-info }
Next, we need to connect our JavaScript to this HTML. To do that, we’ll add a `<script>` tag right before the closing `</body>` tag. The script should have `type="module"` and `src="popup.js"`.

Using `type="module"` allows us to use modern JavaScript features such as `import` and `export` statements. The `src` attribute tells the browser to load the logic from our `popup.js` file — the place where all our “behind-the-scenes” functionality will live. Later, we’ll also update the `innerHTML` of the `<p>` tag to contain a downloadable link once the file has been processed.

> Make sure to include the script tag at the bottom of the body or else your JavaScript will not work on your HTML elements.
{: .prompt-warning }

Your `index.html` should finally look like this:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Syllabus to CSV</title>
</head>
<body>
    <h1>MasterList</h1>
    <input type="file" id="file-upload" name="filename">
    <p>Click on the File to download it:<p>

    <script type="module" src="popup.js"></script>
</body>
</html>
```


## Handle file uploads

Now that we’ve set up a file input, let’s write the JavaScript needed to handle uploaded files and prepare them for processing. Create a file in the root directoy named `popup.js`.

We’ll be using `document.getElementById()` to grab our file input and attach an event listener that runs every time a user selects a file.

```js
document.getElementById('file-upload').addEventListener('change', async () => {
    // all the lines below will go inside here
});
```
{: file="popup.js" }
{: .nolineno }
Here, we’re listening for the "change" event on the file input id (or whatever id you used) — this fires whenever a user picks a file.
```js
async () => {}
```
{: file="popup.js" }
{: .nolineno }
Async allows us to use await inside the function. Since we'll likely send the file to an external API (which takes time), we want to pause and wait for the response before moving on — this keeps the code clean and readable.

Without async, we’d have to use .then() chains, which are harder to manage.

Arrow function syntax (() => {}) is a modern way to write functions in JavaScript. It's short, clean, and avoids creating its own this context — which works well here since we don’t need to refer to the event handler’s context directly.

> **Tip:** In short we use async () => {} to write cleaner, more modern code that lets us easily work with APIs that take time to respond.
{: .prompt-info }
```js
const fileUploaded = this.files.item(0);
```
{: file="popup.js" }
{: .nolineno }
This line grabs the first file the user selected. Since we’re only supporting one file at a time, we access the file at index 0.

Below you should create your own safety check to check if fileUploaded is null. If it is, we want to return.

```js
const form = new FormData();
form.append('purpose', 'ocr');
form.append('file', new File([fileUploaded], `${fileUploaded.name}`));
```
{: file="popup.js" }
{: .nolineno }

Here, we create a FormData object to prepare for sending the file to an external API.

FormData works like a key-value map that you can send with fetch() for things like file uploads.

We add two things:

A purpose field — this is useful if your API requires it (in this case, to label it for OCR processing).

The actual uploaded file, wrapped in a new File object.

> **Note:**  Wrapping the file again with new File([...]) is optional but helpful if you want to manipulate the name or metadata before sending.
{: .prompt-info }

> Important: All of these lines (fileUploaded, if (fileUploaded == null), and the FormData block) should be written inside the event listener function — directly with the async () => {} function.
{: .prompt-danger }

This is the foundation of getting the syllabus file from the user and preparing it for conversion.

At this point your code should look similar to this.
```js
document.getElementById('file-upload').addEventListener('change', async () => {
    const fileUploaded = this.files.item(0);
    if(fileUploaded == null){
        return;
    }
    const form = new FormData();
    form.append('purpose', 'ocr');
    form.append('file', new File([fileUploaded], `${fileUploaded.name}`));
```
{: file="popup.js" }
{: .nolineno }

In the next step, we’ll send this FormData to an OCR model for parsing and response.

