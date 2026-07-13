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

We will be using VS Code in this tutorial but you can use any IDE, you can download VS Code from [visualstudio.com](https://code.visualstudio.com/){:target="\_blank"}. 

{% include embed/youtube.html id='KMxo3T_MTvY' %}

**Install the Live Server extension in Visual Studio Code:** open the Extensions marketplace (usually by clicking on the four squares icon on the left sidebar) and search for "Live Server" by Ritwick Dey. Click "Install".

[Tips and Tricks for VS Code](https://www.youtube.com/watch?v=ifTF3ags0XI&t=343s&ab_channel=Fireship){:target="\_blank"}

## Download the starter code 

For version control, we will use GitHub. If you're not familiar, GitHub is a platform for version control and collaboration, allowing developers to manage, share, and track changes to code efficiently, making teamwork and project management easier. Like Google Drive for programmers.

{% include embed/youtube.html id='hwP7WQkmECE' %}

Here is a more in-depth breakdown oh what GitHub is and how it works [Introduction to GitHub](https://digital.gov/resources/an-introduction-github/){:target="\_blank"}.

**If you don't have it already, download GitHub desktop:** [desktop.github.com/download](https://desktop.github.com/download/){:target="\_blank"}.

**To Fork the code:** 

- Go to [github.com/aidankeighron/Falling-Sand-Tutorial](https://github.com/aidankeighron/Falling-Sand-Tutorial){:target="\_blank"}
- Click the "Fork" button in the top right corner. (This will create a copy of the project in your own GitHub account)
- Then open your newly created fork and click the green "Code" dropdown
- Select "Open with GitHub Desktop"

> A `fork` is a personal copy of a codebase where you can make changes without affecting others.
{: .prompt-info }

# Running the project

Now let's get the project running in your browser.

- **Open the forked project in VS Code:** Go to "File" -> "Open Folder" and select the directory where you cloned your forked repository.
- **Start Live Server:** Open the `index.html` file in VS Code. Right-click anywhere in the file and select "Open with Live Server".
- **Open in a new tab (if it didn't automatically):** Live Server will usually open the webpage in your default browser. If it doesn't, you should see a message in the VS Code status bar at the bottom indicating the port number (e.g., "Port: 5500"). Open a new tab in your browser and navigate to http://127.0.0.1:5500/ (or the port number shown in VS Code).
- 
You should now see a webpage with the title "Falling Sand" and a blank rectangle (the canvas) in the center.
