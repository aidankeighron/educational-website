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
> **TASK 4:** Implement `async function JsonToCSV(markdownExport)` to `POST` the combined markdown to Gemini's `generateContent` endpoint and request assignment extraction in CSV format.
{: .prompt-warning }

> **NOTE:** The more specific and clear your prompt is, the better your results will be. Use `POST` to the Gemini endpoint (`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-lite:generateContent?key=YOUR_API_KEY`), instruct Gemini on exact column headers (`Due Date, Class, Assignment Name, Assignment Type, Checkbox`), and request pure CSV data without markdown ticks. Google often updates available free tier models; check [Google AI Pricing](https://ai.google.dev/gemini-api/docs/pricing) if you need to substitute another free tier model.
{: .prompt-info }

