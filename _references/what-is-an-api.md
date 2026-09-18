---
title: "What is an API?"
description: "A quick, generic explanation of APIs and HTTP methods before you start calling one in a tutorial."
layout: reference
---

Before calling any external API in a tutorial, let's take a quick step back and understand **what an API actually is**.

An **API (Application Programming Interface)** is a way for two programs to talk to each other. Think of it as a bridge that allows two different pieces of software to communicate — often that means your code (running in the browser or on a server) talking to an external service over the internet.

> Think of it like placing an order at a restaurant: you (the client) tell the waiter (the API) what you want, and the waiter brings it from the kitchen (the server). You don't need to know how the kitchen works — just how to place an order properly.
{: .prompt-info }

## HTTP methods

Most APIs you'll work with are **REST APIs**, which are the standard way web applications communicate over the internet. They use HTTP methods — the same methods your browser uses to fetch web pages. The two most common are:

- **GET** — when you want to retrieve data from a source (like reading a webpage).
- **POST** — when you want to send new data to a source (like submitting a login form).

There are also **PUT** and **DELETE**, which you'll meet in tutorials that update or remove data.

## A real-world example

Imagine you want to build a *Pokémon Information App*.

The hard way to build this app would be manually researching and typing out the stats for all 1,000+ Pokémon into your own database. **This is where an API saves the day**. There is a free service called `PokeAPI` that already has all this data. You just have to ask for it!

Here is how the interaction works:

1. **The Request:** You send an HTTP request to the API asking for specific data (e.g., "Give me the stats for Pikachu").
2. **The Processing:** The API server receives your request, finds Pikachu's data in its database, and formats it.
3. **The Response:** The server sends the data back to you in a format your code can read, usually **JSON** (JavaScript Object Notation).

In code, making that request looks like this:

```jsx
const response = await fetch('https://pokeapi.co/api/v2/pokemon/pikachu');
const apiData = await response.json();
```

And the `apiData` you get back will look something like this:

```json
{
 "name": "pikachu",
 "height": 4,
 "weight": 60,
 "types": [
   { "type": { "name": "electric" } }
 ]
}
```

> **QUESTION:** Looking at the JSON data above, how is a JSON object similar to a standard JavaScript object? If you wanted to get the Pokemon's weight from `apiData`, what code would you write?
{: .prompt-tip }

After you get this information back, you can parse it and use it however you'd like.

If you want a deeper visual explanation, check out [this 3-minute video on APIs](https://www.youtube.com/watch?v=s7wmiS2mSXY&t=33s).

## Helpful videos

- [**What is an API?** (by Simply Explained)](https://www.youtube.com/watch?v=ByGJQzlzxQg&t=9s)
  *This video explains APIs using real-world analogies — perfect if you're just starting out.*

- [**4 Most Important HTTP Requests That Can Be Made to an API**](https://www.youtube.com/watch?v=tkfVQK6UxDI)
  *This breaks down the core HTTP methods you'll use when working with APIs: GET, POST, PUT, and DELETE.*

## What is an API key?

An API key is like a password that allows your project to communicate with a third-party service. It tells the API who you are and whether you're allowed to use it.

Think of it like a secret access badge — you'll need one to send requests and get a response back from the service.

> Because an API key acts as a password, it must never be committed to your repository. See [Environment Variables & Secret Safety]({{ '/references/environment-variables/' | relative_url }}) for how to keep it out of version control.
{: .prompt-warning }
