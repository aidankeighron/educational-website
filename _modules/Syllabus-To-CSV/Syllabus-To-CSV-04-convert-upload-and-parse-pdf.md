---
title: "Convert Upload and Parse PDF"
parent_post: Syllabus-To-CSV
module_number: 4
layout: module
media_subpath: /assets/tutorials/csv
---

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
{: file="popup.js" }
{: .nolineno }
Let’s break it down step-by-step:
### The comment block at the top
```js
/**
 * Convert a PDF to a JSON object
 * 
 * @param {FormData} form 
 * @returns {Promise<Object>}
 */
```
{: file="popup.js" }
{: .nolineno }
This is a JSDoc-style comment, which is a great practice even in beginner projects. It tells other developers (or future you):

- What this function does
- What kind of argument it expects (FormData)
- What it returns (a promise that resolves to a JSON object)

> Writing clear comments like this helps others understand your code quickly and makes your project easier to maintain or expand in the future.
{: .prompt-info }

###  Fetch
```js
fetch('https://api.mistral.ai/v1/files', { ... })
```
{: file="popup.js" }
{: .nolineno }
This is the URL of Mistral’s file upload API. When we call this, we’re telling Mistral:

> “Hey, I want to upload a file for OCR processing.”
{: .prompt-info }

```js
method: 'POST'
```
{: file="popup.js" }
{: .nolineno }
This tells the API we want to send data (in this case, the file).
There are other methods like GET, PUT, and DELETE, but POST is most common for sending form or file data.

### Headers
```js
headers: { "Authorization": "Bearer ... " }
```
{: file="popup.js" }
{: .nolineno }
APIs often require authentication — this is how they know who you are.

The "Authorization" header tells the API,

"Here’s my API key — please allow me to use your service."

"Bearer" is the standard keyword used to pass tokens securely.

You should already have your API key stored in `hidden.js`, and here we’re inserting it using backticks and ${} for string interpolation.

### Body
```js
body:form
```
{: file="popup.js" }
{: .nolineno }
This is the actual file upload!
We’re sending the FormData object we created earlier (which includes the PDF file) as the body of the request.
```js
await uploadedPDF.json()
```
{: file="popup.js" }
{: .nolineno }

Once Mistral finishes processing the file, it sends back a response — usually in JSON format.

> If you **don’t** call `.json()` and just look at the `response` object itself, you’ll get a **network response object**, not the actual data.
{: .prompt-danger }

We call .json() on the response to convert it into an object we can work with in JavaScript.

### What does Mistral send back?
It doesn’t send back the converted syllabus — not yet.

Instead, it responds with file metadata, like this:

```json
{
  "id": "file-abc123",
  "status": "uploaded",
  "filename": "syllabus.pdf",
  "created_at": "2024-03-23T15:12:00Z"
}
```
{: .nolineno }
This response tells us:

The upload was successful!

We now have a file ID that we can use to request a signed download URL in the next step

The OCR processing hasn’t happened yet — we’ll request it next!

> Important: This function does not do OCR yet. It only uploads the file and gives back an ID.
We'll use this ID in a follow-up request to get the downloadable link and send that to Mistral's OCR model.
{: .prompt-info }

### Upload PDF to get URL

Now that we’ve uploaded the file, Mistral gave us a **file ID** in the response. We’re going to use that ID to request a **signed file URL** — a secure, temporary link to download or reference the uploaded file.

### Make the API Call

Use the `fetch()` function to make a **GET request** to this endpoint: https://api.mistral.ai/v1/files/FILE_ID/url?expiry=24

> Replace `FILE_ID` with the ID you received from the previous step (`PDFJson.id`)
{: .prompt-info }

This tells Mistral:  

> “Please give me a temporary link to access the file I just uploaded.”
{: .prompt-info }

The `expiry=24` part means the link will only work for **24 hours**.

### Headers You’ll Need

Your request should include a `headers` object with the following:

```js
headers: {
  "Accept": "application/json",
  "Authorization": `Bearer ${mistralApiKey}`
}
```
{: file="popup.js" }
{: .nolineno }
> **QUESTION:** What does the header `"Accept": "application/json"` communicate to the API server? What might happen if a client doesn't specify which content format it expects back?
{: .prompt-tip }

> **TASK 1:** Use `fetch()` to make a `GET` request to `https://api.mistral.ai/v1/files/FILE_ID/url?expiry=24` (replacing `FILE_ID` with `PDFJson.id`), pass the required `headers`, and extract the signed download URL using `.json()`.
{: .prompt-warning }

Once you’ve done that, you’ll have access to a temporary URL like:
```json
{
  "url": "https://cdn.mistral.ai/files/abc123/syllabus.pdf?token=..."
}
```
{: .nolineno }
We’ll use that URL in the next step when we send the file to Mistral’s OCR model for analysis!

> Hint: Store the result in a variable like responseJSON, then access the URL with responseJSON.url
{: .prompt-info }


## Parse PDF to Markdown

Now that you’ve obtained a **temporary URL** to the uploaded file, it’s time to send that file to Mistral’s OCR model and get back structured text.

### Your Turn: Make the OCR API Call

You’re going to use `fetch()` again — this time to **POST** the signed URL to Mistral’s OCR endpoint.

**API Endpoint:** https://api.mistral.ai/v1/ocr

In your request, you’ll need these headers:

```js
headers: {
  "Content-Type": "application/json",
  "Authorization": `Bearer ${mistralApiKey}`
}
```
{: file="popup.js" }
{: .nolineno }
The Body (What You’re Sending)
Before we send the body, we need to convert our JavaScript object into a string using JSON.stringify().

> **QUESTION:** Why do web APIs expect serialized JSON strings (via `JSON.stringify()`) in HTTP request bodies instead of raw in-memory JavaScript objects?
{: .prompt-tip }

> **TASK 2:** Use `fetch()` with method `'POST'` to send the JSON-stringified document payload to `https://api.mistral.ai/v1/ocr`, pass the authentication headers, and extract the result using `.json()`.
{: .prompt-warning }

