---
title: "Basic JavaScript Knowledge"
parent_post: Falling-Sand
module_number: 2
layout: module
media_subpath: /assets/tutorials/falling sand
---

This section will cover some fundamental JavaScript concepts that we'll be using in this tutorial. If you are already comfortable with JavaScript, feel free to skip to the next section. [w3schools](https://www.w3schools.com/js/default.asp){:target="\_blank"} is a great resource for learning or refreshing your knowledge of any of these concepts.

# HTML DOM (accessing HTML DOM using `getElementById`)

The HTML Document Object Model (DOM) represents the structure of your HTML document as a tree of objects. JavaScript can interact with this tree to dynamically change the content and behavior of your webpage.

{% include embed/youtube.html id='ok-plXXHlWw' %}

**Accessing HTML elements in JavaScript:**

`getElementById` is a JavaScript method that allows you to access a specific HTML element by its id attribute. In our `index.html` file, you'll find elements with IDs like canvas, speedRange, and clear-button. We can access these elements in our JavaScript code like this:

**Example (you don't need to add this code)**

```js
const canvasElement = document.getElementById('canvas');
const speedSliderElement = document.getElementById('speedRange');
const clearButtonElement = document.getElementById('clear-button');

console.log(canvasElement); // This will log the canvas HTML element to the console.
```

Further reading: [w3schools HTML DOM](https://www.w3schools.com/js/js_htmldom.asp)

# Event listeners

Event listeners allow you to respond to specific events that happen on your webpage, such as a user clicking a button or moving their mouse. We attach event listeners to HTML elements using JavaScript.

Here's an example of how we can add an event listener to our "Clear Screen" button:

**Example (you don't need to add this code)**

```js
const clearButtonElement = document.getElementById('clear-button');

// The arrow syntax is called a lambda expression
clearButtonElement.addEventListener('click', () => {
    // When the user clicks on the clearButtonElement, this code will run
    console.log('Clear button clicked!');
});
```

More info on [lambda expressions or (arrow functions)](https://www.w3schools.com/js/js_arrow_function.asp).

In this code:

- We get the button element using its ID.
- We use the `addEventListener` method to attach a function to the `'click'` event.
- The function inside `addEventListener` will be executed every time the button is clicked.

Further reading: [w3schools Event Listener](https://www.w3schools.com/js/js_htmldom_eventlistener.asp)
