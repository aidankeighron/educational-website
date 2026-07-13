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

### What is an API?

Before we use Mistral OCR, let’s take a quick step back and understand **what an API actually is**.

An **API (Application Programming Interface)** is a way for two programs to talk to each other. In our case, we’ll be using JavaScript to talk to an external AI service (Mistral) — and that conversation happens through an API.

> Think of it like placing an order at a restaurant: you (the client) tell the waiter (the API) what you want, and the waiter brings it from the kitchen (the server). You don’t need to know how the kitchen works — just how to place an order properly.
{: .prompt-info }

### Helpful Videos

- [**What is an API?** (by Simply Explained)](https://www.youtube.com/watch?v=ByGJQzlzxQg&t=9s)  
  *This video explains APIs using real-world analogies — perfect if you're just starting out.*

- [**4 Most Important HTTP Requests That Can Be Made to an API**](https://www.youtube.com/watch?v=tkfVQK6UxDI)  
  *This breaks down the core HTTP methods you'll use when working with APIs: GET, POST, PUT, and DELETE.*

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
If you’re using Git, be sure to add `hidden.js` to your `.gitignore`.
{: .prompt-danger }

### What is a .gitignore file?
When you use Git to track your project’s files (like code, images, config files), you don’t always want everything to be tracked or pushed to GitHub. That’s where `.gitignore` comes in.

> A `.gitignore` file tells Git: Ignore these files. Don’t include them in version control or upload them to GitHub.

This is really helpful for:

- Sensitive files (like API keys in `hidden.js`)
- Build folders (dist/, node_modules/, etc.)
- System files (like .DS_Store on macOS or Thumbs.db on Windows)

### How it works
If a file or folder matches a rule in `.gitignore`, Git will pretend it doesn’t exist.
It won’t track changes to it, and it won’t push it to a remote repo like GitHub.  

### How to add something to .gitignore
Just open the `.gitignore` file in your project root (or create one if it doesn’t exist), and add the filename or folder you want to ignore.

For example:

```gitignore
# Ignore API key file
hidden.js

# Ignore all files in node_modules/
node_modules/
```
{: file=".gitignore" }
{: .nolineno }

Now Git will skip these files when committing or pushing your code — keeping things secure and clean.

> Best practice: Always add secret files like `hidden.js` to `.gitignore` before uploading your project to GitHub.
{: .prompt-tip }

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

