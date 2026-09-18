---
title: "Getting Your First API Key"
parent_post: Syllabus-To-CSV
module_number: 3
layout: module
media_subpath: /assets/tutorials/csv
---

## Getting Your first API Key

Before we can send our syllabus to an AI model, we need an API key to authenticate with [Mistral OCR](https://mistral.ai/news/mistral-ocr), the model we’ll be using to process and extract text from PDF files.

### What is Mistral OCR?

Mistral OCR is a powerful AI model that can extract structured information from scanned documents, including PDFs — which is exactly what we need for turning a syllabus into a list of assignments.

> Learn more: [Mistral OCR announcement](https://mistral.ai/news/mistral-ocr)
{: .prompt-info }

> New to APIs? Check out our [What is an API?]({{ '/references/what-is-an-api/' | relative_url }}) background page before continuing.
{: .prompt-info }

### What is an API Key?

An API key is like a password that allows your project to communicate with a third-party service (in this case, Mistral). It tells the API who you are and whether you’re allowed to use it.

Think of it like a secret access badge — you’ll need one to send your file and get a response from Mistral.

### Step 1: Get your Mistral API Key

1. Go to [https://console.mistral.ai/api-keys](https://console.mistral.ai/api-keys)
2. Log in or create an account
3. If this is your first time, you’ll be prompted to choose an API plan — make sure to select the free one
4. Click **“Create API Key”**
5. Copy the key — it will look something like:  mistral-key-abc1234567890
   
> Important: If you skip selecting a plan, your API key won’t be usable yet. Be sure to select the free tier after signing up so you can continue with the tutorial.
{: .prompt-warning }   

### Step 2: Create a `hidden.js` file

To keep your API key separate from your main code (and avoid accidentally uploading it), let’s store it in a new file.

Create a file called: `hidden.js`

And inside it, write:

```js
const mistralApiKey = "your-mistral-api-key-here";
const geminiApiKey = "your-gemini-api-key-here"; // for using Gemini later

export default {
  mistralApiKey,
  geminiApiKey
};
```
{: file="hidden.js" }
{: .nolineno }

> Never commit this file to GitHub!
If you’re using Git, be sure to add `hidden.js` to your `.gitignore` file.
{: .prompt-danger }

> New to `.gitignore`? Check out our [Git & GitHub Basics]({{ '/references/git-github-basics/' | relative_url }}) background page for what it is and how to use it.
{: .prompt-info }

### Step 3: Import your API keys
In your `popup.js`, import them like this:
```js
import apiKeys from "./hidden.js";
const mistralApiKey = apiKeys.mistralApiKey;
const geminiApiKey = apiKeys.geminiApiKey;
```
{: file="popup.js" }
{: .nolineno }

You're now ready to securely connect to Mistral and begin sending files for parsing! Next up: we’ll write the code that sends our FormData to the Mistral OCR API!

