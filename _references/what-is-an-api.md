---
title: "What is an API?"
description: "A quick, generic explanation of APIs and HTTP methods before you start calling one in a tutorial."
layout: reference
---

Before calling any external API in a tutorial, let's take a quick step back and understand **what an API actually is**.

An **API (Application Programming Interface)** is a way for two programs to talk to each other. Often that means your code (running in the browser or on a server) talking to an external service over the internet.

> Think of it like placing an order at a restaurant: you (the client) tell the waiter (the API) what you want, and the waiter brings it from the kitchen (the server). You don't need to know how the kitchen works — just how to place an order properly.
{: .prompt-info }

## HTTP methods

Most APIs you'll work with are **REST APIs**, the most common type of API. They communicate via HTTP methods, most commonly GET and POST (there's also PUT and DELETE):

- **GET** — when you want to get data from the source.
- **POST** — when you want to send data to the source.

## Example: fetching data from an API

Imagine you want to build a *Pokemon Information App*. The hard way to make this app is to collect every single bit of information about every single Pokemon yourself. This is where an API makes the process much easier — there's an API called `PokeAPI` where you send a request for Pokemon data, and it sends it back to you.

Here's how it works:

- You send an HTTP request.
- You specify what you want in the request — for example, everything about Pikachu.
- The other end of the API processes the request, gathers the information about Pikachu, and puts it in a JSON file so you can understand it.
- It sends back the information you requested, and now you have everything you needed without collecting any data yourself!

As a programmer, it would look like this. You send a request like:

```jsx
const apiData = await fetch('https://pokeapi.co/api/v2/pokemon/pikachu')
```

And get something back that looks like this:

```jsx
// Note: This is an example, not what PokeAPI will actually send
{
 "name": "pikachu",
 "height": 4,
 "weight": 60,
 "types": [
   { "type": { "name": "electric" } }
 ]
}
```

After you get this information back, you can parse it and use it however you'd like.

[Here is another example](https://www.youtube.com/watch?v=s7wmiS2mSXY&t=33s) if you're struggling a bit to understand.

## Helpful videos

- [**What is an API?** (by Simply Explained)](https://www.youtube.com/watch?v=ByGJQzlzxQg&t=9s)
  *This video explains APIs using real-world analogies — perfect if you're just starting out.*

- [**4 Most Important HTTP Requests That Can Be Made to an API**](https://www.youtube.com/watch?v=tkfVQK6UxDI)
  *This breaks down the core HTTP methods you'll use when working with APIs: GET, POST, PUT, and DELETE.*

## What is an API key?

An API key is like a password that allows your project to communicate with a third-party service. It tells the API who you are and whether you're allowed to use it.

Think of it like a secret access badge — you'll need one to send requests and get a response back from the service.
