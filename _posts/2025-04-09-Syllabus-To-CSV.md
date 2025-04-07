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

> Make sure you reload your extension after changing anything or you will not see the changes
{: .prompt-info }

## Create a basic UI

### HTML

### CSS

## Handle file uploads

## Convert upload to PDF

### Upload PDF

### Parse into JSON

## Parse upload into assignment list

### Sending upload to gemini

### Downloading the file

## Extending your extension