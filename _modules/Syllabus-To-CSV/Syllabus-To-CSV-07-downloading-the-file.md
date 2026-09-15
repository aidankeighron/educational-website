---
title: "Downloading the File"
parent_post: Syllabus-To-CSV
module_number: 7
layout: module
media_subpath: /assets/tutorials/csv
---

## Downloading the file
Now that Gemini has returned your assignment list in CSV format, the final step is to let the user download it!

We’ll do this by programmatically creating a downloadable file in the browser using JavaScript.

Here’s the function you’ll use:
```
function createFileAndDownload(filename, content) {
    const blob = new Blob([content], { type: 'text/plain' });
    const url = URL.createObjectURL(blob);
    const link = document.createElement('a');
    link.href = url;
    link.download = filename;
    document.body.appendChild(link);
    const p = document.createElement('p');
    p.innerHTML = filename;
    link.append(p);
}
```
{: file="popup.js" }
{: .nolineno }

### How does this work?
- new Blob([content]): This creates a binary object (Blob) from your text. Think of it like a fake file we can give to the browser.

- URL.createObjectURL(blob): Generates a temporary download link from the Blob.

- document.createElement('a'): We create an anchor (<a>) tag and set its href to our blob URL.

- link.download = filename: This tells the browser what to name the downloaded file.

- Finally, we append the link (with a label) so the user can click and download it.

You can now call this function like so:

```js
createFileAndDownload("assignments.csv", geminiResponse);
```
{: file="popup.js" }
{: .nolineno }

If your response string includes some extra characters (like Markdown code block markers), make sure to clean it up first:
```js
const cleaned = geminiResponse.slice(6).slice(0, -3);
createFileAndDownload("assignments.csv", cleaned);
```
### Finishing our Event Listener
By the end of this tutorial, your full addEventListener function should look something like this:
```js
document.getElementById('file-upload').addEventListener('change', async (event) => {
    // Get fileUploaded, returns file object at index 0
    const fileUploaded = event.target.files[0];
    if (fileUploaded == null) {
        return;
    }

    // Create form object for PDF send to OCR API
    const form = new FormData();
    form.append('purpose', 'ocr');
    form.append('file', new File([fileUploaded], `${fileUploaded.name}`));

    // Send to Mistral and get structured markdown
    let ocrJson = await PDFToJson(form);

    // Combine all markdown content into one string
    let markdownExport = "";
    for (const element of ocrJson.pages) {
        markdownExport += element.markdown + " ";
    }

    // Send combined markdown to Gemini for CSV generation
    const geminiJson = await JsonToCSV(markdownExport);
    const geminiResponse = geminiJson.candidates[0].content.parts[0].text;

    // Download the result as a .csv file
    createFileAndDownload("downloadable.csv", geminiResponse.slice(6).slice(0, -3));
});
```
{: file="popup.js" } 
{: .nolineno }
> That’s it! You’ve now built a full Chrome extension that lets users upload a syllabus, extracts all assignments using AI, and downloads the results as a clean CSV file. 
{: .prompt-success }

## Completion & Discussion Checklist

Before joining the group discussion or concluding this tutorial, ensure you have completed the tasks, investigated the bugs, and are ready to discuss the questions below:

<details markdown="1">
<summary>Click to expand Completion & Discussion Checklist (7 Items)</summary>

| # | Type | Item | Prompt Preview |
| :-: | :--- | :--- | :--- |
| 1 | Bug Hunt | Arrow Function `this.files` Context | If you test this handler, DevTools logs `TypeError: Cannot read properties of undefined (reading 'item')`. Why is `this.files` undefined in an arrow function, and how does `event.target.files[0]` fix it? |
| 2 | Question | HTTP `Accept` Header Negotiation | What does the header `"Accept": "application/json"` communicate to the API server? What might happen if a client doesn't specify which format it expects back? |
| 3 | Question | JSON String Serialization | Why do web APIs expect serialized JSON strings (via `JSON.stringify()`) in HTTP request bodies instead of raw in-memory JavaScript objects? |
| 4 | Task | Fetch Signed Download URL | Use `fetch()` to make a `GET` request to `https://api.mistral.ai/v1/files/FILE_ID/url?expiry=24`, pass the required headers, and extract the signed download URL using `.json()`. |
| 5 | Task | Execute Mistral OCR Request | Use `fetch()` with method `'POST'` to send the JSON-stringified document payload to `https://api.mistral.ai/v1/ocr`, pass headers, and extract the result using `.json()`. |
| 6 | Task | Aggregate Multi-Page Markdown | Loop through `ocrJson.pages` and concatenate the `markdown` property from each page into a single combined Markdown string. |
| 7 | Task | Gemini Structured CSV Generation | Implement `async function JsonToCSV(markdownExport)` to `POST` the combined markdown to Gemini's endpoint and request assignment extraction in CSV format. |

</details>



