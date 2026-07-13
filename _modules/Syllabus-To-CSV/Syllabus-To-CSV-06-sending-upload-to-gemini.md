---
title: "Sending Upload to Gemini"
parent_post: Syllabus-To-CSV
module_number: 6
layout: module
media_subpath: /assets/tutorials/csv
---

## Sending upload to gemini

Now that you've combined all of your syllabus content into a single Markdown string, you're ready to send it to an AI model — in this case, **Google Gemini** — to extract your assignments and return them in a clean CSV format.

### Step 1: Get Your Gemini API Key

To use Gemini, you'll need to create an API key from Google’s developer console.

1. Go to [https://ai.google.dev/gemini-api/docs/api-key](https://ai.google.dev/gemini-api/docs/api-key)
2. Sign in with your Google account
3. Click **Create API Key**
4. Copy the key and store it safely — we’ll use this in our fetch request

Just like we did with Mistral, you should store this key in your `hidden.js` file:

```js
const geminiApiKey = "your-gemini-api-key-here";
```
{: file="hidden.js" }
{: .nolineno }
Your Task: Send the Markdown to Gemini
Here’s what you need to do:

Create this function
```js
async function JsonToCSV(markdownExport) {}
```
{: file="popup.js" }
{: .nolineno }
Use fetch() to send a POST request to this Gemini endpoint:


`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=YOUR_API_KEY`

Replace YOUR_API_KEY with your Gemini key (preferably from `hidden.js`).

In the headers, include:

```
"Content-Type": "application/json"
```
{: file="popup.js" }
{: .nolineno }
In the body of the request:
- Use JSON.stringify() to convert your request body to JSON
- Create a prompt asking Gemini to extract assignments from the Markdown you created
- Ask for a CSV format with these columns:
- Due Date
- Class
- Assignment Name
- Assignment Type (from: Homework, Reading, Project, Exam)
- Checkbox

Make sure to include your entire markdownExport inside the prompt using a template string (${}).

> Tip: The more specific and clear your prompt is, the better your results will be. You’re essentially saying:
"Hey Gemini, here’s a syllabus in Markdown. Can you pull out the assignments and return them in a neat table?"
{: .prompt-info }

Your goal here is to get back a Gemini response containing a CSV-formatted list of assignments from your syllabus.

We’ll use this response in the next step to create a downloadable .csv file the user can save!

