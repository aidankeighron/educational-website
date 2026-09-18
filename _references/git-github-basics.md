---
title: "Git & GitHub Basics"
description: "What Git and GitHub are, how forking works, and how to keep secrets out of your repo with .gitignore."
layout: reference
---

## What is GitHub?

For version control, most of these tutorials use GitHub. If you're not familiar, GitHub is a platform for version control and collaboration, allowing developers to manage, share, and track changes to code efficiently, making teamwork and project management easier. Like Google Drive for programmers.

{% include embed/youtube.html id='hwP7WQkmECE' %}

Here is a more in-depth breakdown of what GitHub is and how it works: [Introduction to GitHub](https://digital.gov/resources/an-introduction-github/){:target="\_blank"}.

## Forking a project

**If you don't have it already, download GitHub desktop:** [desktop.github.com/download](https://desktop.github.com/download/){:target="\_blank"}.

To fork a project on GitHub:

- Go to the project's GitHub page
- Click the "Fork" button in the top right corner (this creates a copy of the project in your own GitHub account)
- Then open your newly created fork and click the green "Code" dropdown
- Select "Open with GitHub Desktop"

> A `fork` is a personal copy of a codebase where you can make changes without affecting others.
{: .prompt-info }

## What is a `.gitignore` file?

When you use Git to track your project's files (like code, images, config files), you don't always want everything to be tracked or pushed to GitHub. That's where `.gitignore` comes in.

> A `.gitignore` file tells Git: Ignore these files. Don't include them in version control or upload them to GitHub.

This is really helpful for:

- Sensitive files (like API keys)
- Build folders (dist/, node_modules/, etc.)
- System files (like .DS_Store on macOS or Thumbs.db on Windows)

### How it works

If a file or folder matches a rule in `.gitignore`, Git will pretend it doesn't exist. It won't track changes to it, and it won't push it to a remote repo like GitHub.

### How to add something to .gitignore

Just open the `.gitignore` file in your project root (or create one if it doesn't exist), and add the filename or folder you want to ignore.

For example:

```gitignore
# Ignore an API key file
hidden.js

# Ignore all files in node_modules/
node_modules/
```
{: file=".gitignore" }
{: .nolineno }

Now Git will skip these files when committing or pushing your code — keeping things secure and clean.

> Best practice: Always add secret files (like ones containing API keys) to `.gitignore` before uploading your project to GitHub.
{: .prompt-tip }
