---
title: "Setup the Project"
parent_post: FocusTube
module_number: 1
layout: module
media_subpath: /assets/tutorials/focustube
---

## Setup the project

You will be making your own project instead of forking a repository. Making a NextJS app is incredibly easy. 

Open a terminal. In the terminal, go to the folder you want your NextJS project to be in, then run this command:

```console
$ npx create-next-app@latest
```

You will be given multiple prompts on how you want your project to be. Below are our supported settings for the tutorial.

```console
✔ What is your project named? … my-app
✔ Would you like to use TypeScript? … No
✔ Which linter would you like to use? » ESLint
✔ Would you like to use Tailwind CSS? … Yes
✔ Would you like your code inside a `src/` directory? … Yes
✔ Would you like to use App Router? (recommended) … Yes
√ Would you like to customize the import alias (`@/*` by default)? ... No
√ Would you like to include AGENTS.md to guide coding agents to write up-to-date Next.js code? ... Yes
```

> **QUESTION:** While we've provided the recommended setup options for this tutorial, it's highly encouraged to know what they actually do! Take a moment to research: What exactly is ESLint? What is the difference between the Next.js `App Router` and the older `Pages Router`?
{: .prompt-tip }

Now you will now have a folder for your project. I named mine **/my-app**, as seen above. <span style="text-decoration: underline;" title="type 'cd <file path>' into your terminal to change your working directory">cd</span> into that folder and run:

```console
$ npm run dev
```

This will start up your app and you will can visit it [here at localhost:3000](http://localhost:3000/)

> **NOTE:** When you change your project, this will automatically update the website for you. There is no need to keep terminating and restarting the program.
{: .prompt-info }

Now you are ready to start the tutorial!

Take a look at your file tree. It should look something like this:

```
my-app/
├── node_modules/
├── public/
├── src/
│   └── app/
│       ├── favicon.ico
│       ├── globals.css
│       ├── layout.js
│       └── page.js
├── .gitignore
├── eslint.config.mjs
├── jsconfig.json
├── next.config.mjs
├── package-lock.json
├── package.json
├── postcss.config.js
└── README.md
```

Feel free to check out **page.js** in your app folder, this is the **home webpage** in your NextJS app, which is a good transition into the first topic.

> **QUESTION:** There are some new files in our project. What are `.gitignore`, `node_modules`, and `package.json` for? What happens when you delete `package-lock.json`?
{: .prompt-tip }
