---
title: "Setup and Running the Project"
parent_post: Falling-Sand
module_number: 1
layout: module
media_subpath: /assets/tutorials/falling sand
---

# Setup

First, you'll need to get the project code and set up your development environment.

## Set up your IDE 

We will be using VS Code in this tutorial, with the Live Server extension for running the project locally.

> New to VS Code or need to set up Live Server? Check out our [Setting Up VS Code]({{ '/references/vscode-setup/' | relative_url }}) background page.
{: .prompt-info }

## Download the starter code 

For version control, we will use GitHub.

> New to Git/GitHub? Check out our [Git & GitHub Basics]({{ '/references/git-github-basics/' | relative_url }}) background page for what Git/GitHub is and how forking works.
{: .prompt-info }

**If you don't have it already, download GitHub desktop:** [desktop.github.com/download](https://desktop.github.com/download/){:target="\_blank"}.

**To Fork the code:** 

- Go to [github.com/aidankeighron/Falling-Sand-Tutorial](https://github.com/aidankeighron/Falling-Sand-Tutorial){:target="\_blank"}
- Click the "Fork" button in the top right corner. (This will create a copy of the project in your own GitHub account)
- Then open your newly created fork and click the green "Code" dropdown
- Select "Open with GitHub Desktop"

# Running the project

Now let's get the project running in your browser.

- **Open the forked project in VS Code:** Go to "File" -> "Open Folder" and select the directory where you cloned your forked repository.
- **Start Live Server:** Open the `index.html` file in VS Code. Right-click anywhere in the file and select "Open with Live Server".
- **Open in a new tab (if it didn't automatically):** Live Server will usually open the webpage in your default browser. If it doesn't, you should see a message in the VS Code status bar at the bottom indicating the port number (e.g., "Port: 5500"). Open a new tab in your browser and navigate to http://127.0.0.1:5500/ (or the port number shown in VS Code).
- 
You should now see a webpage with the title "Falling Sand" and a blank rectangle (the canvas) in the center.
