---
title: Express Authentication App
author: cuong
date: 2025-07-16 12:00:00 +0700
categories: [ExpressAuth]
layout: tutorial-hub
tutorial_slug: Express-Auth
references: [what-is-an-api, environment-variables]
tags: [TypeScript, Intermediate, Authentication, JWT, Express, MongoDB, React]
description: Learn how to build a secure authentication system using Express, MongoDB, React, and TypeScript
comments: false
pin: true
media_subpath: /assets/tutorials/express-auth
image: /thumb-1920-1358310.png
---

## About the project

Welcome to the Express Authentication tutorial! In this project, you will learn how to build a complete full-stack contact management application with secure user authentication.

![Authentication Demo](/Animation.gif)

**Prerequisites**:

- Basic understanding of how the web works, APIs, sending and receiving requests
- Basic coding skills in JS. React skills are preferable

**What you will learn:**

- How to set up a Node.js backend with Express and TypeScript
- Creating and managing MongoDB database models with Mongoose
- Implementing secure authentication with JWT (JSON Web Tokens)
- Building a React frontend with TypeScript, hooks, and context
- Proper error handling and user feedback with notifications
- Creating protected routes that require authentication

**What you will make:**

By the end of this tutorial, you will have built a fully functional application where users can:

- Register a new account with validation
- Log in securely with JWT authentication
- Create and view contact information
- Experience proper session management with token persistence
- Navigate through a protected application with proper routing

**Further possibilities:**

You can add more to this project:

- Implement refresh/access token
- Add "forget password" field and send email to user to verify
- Add edit/delete functionalities for your contact
- Improve styling
- Use a different database

## Project Overview

Our application consists of two parts:

1. **Backend**: Node.js with Express and TypeScript, connected to MongoDB
2. **Frontend**: React with TypeScript

The application will allow users to:

- Register a new account
- Log in with secure authentication
- View their contacts
- Add new contacts

Let's get started!
