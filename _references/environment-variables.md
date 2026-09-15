---
title: "Environment Variables & Secret Safety"
description: "What environment variables are, why API keys must never be committed, and how .env and .gitignore keep your secrets out of your repository."
layout: reference
---

Several tutorials on this site ask you to get an API key. Before you paste one into a project, it's worth understanding **where secrets should live** and why.

## What are environment variables?

An **Environment Variable** is a configuration value that is set outside of your application's source code. Instead of hardcoding settings (like API URLs, ports, or secret keys) directly into your code, your application reads these values from the running environment at runtime.

This approach is a core software engineering standard (part of the [Twelve-Factor App methodology](https://12factor.net/config)) and offers several key benefits:

- **Separation of Concerns**: Your codebase remains clean and independent of where it runs. The same code works on your developer machine, a test server, or a live production container.
- **Environment-Specific Configs**: You can configure your app to connect to a local mock database during development, and transition to a secure database on your cloud server in production, all without changing a single line of code.
- **Improved Security**: Private tokens, passwords, and API keys are stored outside the source code, meaning they won't get committed to version control.

### The role of `.env` files

When developing locally, setting environment variables inside your system terminal can be tedious. Instead, we use a file named `.env` in the root of the project to store configurations as simple `KEY=VALUE` pairs:

```console
DB_PASSWORD=my_secure_dev_password
EXPO_PUBLIC_API_KEY=your-api-key-here
```

When you start your development server (e.g., via Next.js or Expo), a library like `dotenv` automatically parses this local `.env` file and loads the variables, making them accessible in your code via `process.env.EXPO_PUBLIC_API_KEY`.

## Why is security important?

Hardcoding credentials in your source code and pushing them to repositories like GitHub is one of the most common causes of security breaches.

### The risks of secret leaks

- **Automated Scanning Bots**: Millions of script bots crawl public GitHub commits every second, scanning for patterns resembling AWS credentials, Google Cloud tokens, OpenAI API keys, and database passwords. A key pushed to a public repository can be intercepted and exploited **in under 30 seconds**.
- **Financial Liability**: If a hacker steals your AWS or Google Cloud API key, they can run scripts to spin up thousands of high-performance cloud servers to mine cryptocurrency. This can rack up **tens of thousands of dollars** in charges overnight.
- **Data Breaches & Deletions**: A leaked database password gives attackers full access to view, download, or completely erase user tables, often followed by a ransomware demand.
- **Service Quota Exhaustion**: For public APIs with usage quotas (like Google's YouTube Data API), a leaked key can quickly reach its rate limit, crashing your live app for all legitimate users.

### How `.gitignore` protects you

To prevent secret leaks, we use a special Git configuration file called `.gitignore`. This file sits in the root of your project and lists the names of files and folders that Git should completely ignore.

By adding `.env` to your `.gitignore` file, you ensure that Git will never track or upload your secret keys to your repository:

```console
# local env files
.env
.env*.local
```
{: file=".gitignore" }

Each developer on a team should maintain their own local `.env` file containing their own dev credentials.

> If you have already committed a `.env` file to your repository and then add it to `.gitignore` later, Git will **continue tracking** it. You must run `git rm --cached .env` to untrack it, and **rotate/revoke** the leaked key immediately since it is stored in your commit history.
{: .prompt-warning }

> For more on what `.gitignore` is and how Git tracks files in general, see [Git & GitHub Basics]({{ '/references/git-github-basics/' | relative_url }}).
{: .prompt-info }

## Best practices

To manage secrets safely, keep these best practices in mind:

- **Configure Hosting Dashboards**: When deploying to production platforms like Vercel, Netlify, Render, or Heroku, never upload your `.env` files. Instead, use the platform's Web Dashboard under **Settings > Environment Variables** to input your keys securely. The platform will automatically inject them into your application container at startup.
- **Manage Mobile Secrets (Expo EAS)**: For React Native apps using Expo Application Services (EAS) for builds, configure your production keys as EAS secrets using the command line:

  ```console
  eas secret:create --name EXPO_PUBLIC_API_KEY --value your-prod-api-key
  ```

- **Understand Frontend Exposure**: see the warning below before prefixing anything with `EXPO_PUBLIC_` or `NEXT_PUBLIC_`.
- **Use Template Files**: Commit a template file named `.env.example` to your repository. This file should define all required key names but leave the values blank (e.g. `EXPO_PUBLIC_API_KEY=`). This shows other developers what variables they need to configure locally to get the app running without exposing your actual keys.
- **Immediately Rotate Leaked Keys**: If you suspect a key has been committed or leaked, go to the provider console (e.g., Google Developer Console), revoke the key, and generate a new one immediately.

> Environment variables prefixed with `EXPO_PUBLIC_` (Expo) or `NEXT_PUBLIC_` (Next.js) are explicitly packaged and shipped inside the client-side JavaScript bundle. This means **anyone can inspect your compiled app files or monitor network traffic to read them**.
{: .prompt-danger }

Only use frontend-prefixed environment variables for public, restricted keys (like Google API keys that are rate-limited and locked down in the Google Console). Never put private database passwords or admin tokens in frontend-prefixed variables. For high-security tasks, route client requests through a secure backend proxy server that handles the private credentials safely.
