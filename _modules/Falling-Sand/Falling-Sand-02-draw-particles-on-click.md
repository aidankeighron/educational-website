---
title: "Draw Particles on Click"
parent_post: Falling-Sand
module_number: 2
layout: module
media_subpath: /assets/tutorials/falling sand
---

Now let's start drawing particles on the canvas when you click the mouse.

> This module uses the DOM and event listeners. New to those? Check out our [JavaScript & DOM Basics]({{ '/references/javascript-dom-basics/' | relative_url }}) background page first.
{: .prompt-info }

# Brief overview of how canvas works

The HTML `<canvas>` element is used to draw graphics on a webpage using JavaScript. It's like a blank painting surface that you can control with code. To draw on the canvas, you first need to get its 2D rendering context. This context provides methods for drawing shapes, text, images, and more.

In our `canvas.js` file, you'll see this code at the top:

**Example (you don't need to add this code)**

```js
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext("2d");
```
{: file="canvas.js" }
{: .nolineno }

This code gets the canvas element from our HTML and then gets its 2D rendering context, which we store in the `ctx` variable. We'll use this `ctx` variable to draw our particles.

# Mouse Listeners

In the `canvas.js` file, you'll also find a function called `setUpMouseListeners`:

**Example (you don't need to add this code)**

```js
export function setUpMouseListeners() {
    // On start mouse click
    canvas.addEventListener("mousedown", (event) => {
        isDragging = true;
        mousePosition = {clientX: event.clientX, clientY: event.clientY};
    });
    // On move mouse
    canvas.addEventListener("mousemove", (event) => {
        mousePosition = event;
    });
    // On release mouse click
    canvas.addEventListener("mouseup", (event) => {
        isDragging = false;
    });
}
```
{: file="canvas.js" }
{: .nolineno }

This function sets up three event listeners on our `canvas` element:

- **mousedown:** This event is triggered when you press the mouse button down while the cursor is over the canvas. When this happens, we set the `isDragging` variable to true and store the current mouse coordinates in the `mousePosition` variable.
- **mousemove:** This event is triggered every time you move the mouse cursor while it's over the canvas. We update the `mousePosition` variable with the latest coordinates.
- **mouseup:** This event is triggered when you release the mouse button. We set the `isDragging` variable back to `false`.

In our `main.js` file, inside the `update` function, you'll see how we use these listeners to create particles when the mouse is clicked and dragged:

```js
function update() {
    // Get mouse position
    const {isDragging, mousePosition} = getMouse();
    // If dragging (clicked) and a valid mouse position then create a new particle
    if (isDragging && mousePosition) {
        createParticle(mousePosition);
    }
    // ... rest of the update function
}
```
{: file="main.js" }
{: .nolineno }

# Draw particles on the canvas

Now, let's write the code that actually draws the particles on the canvas. Open the `canvas.js` file and find the `redraw` function. You'll see a `TODO` comment inside the inner for loop. 

> If you looked at how we are creating `grid` you will see that this code does have an error, we will fix it in the next section.
{: .prompt-info }

```js
export function redraw() {
    // Clear previous frame
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    // Loop through all elements in the grid
    for (let row = 0; row < grid.length; row++) {
        for (let col = 0; col < grid[0].length; col++) {
            // 👇 Put the following code here 👇
            const particle = grid[row][col];

            // Get particle color
            ctx.fillStyle = particle.color;
            // Draw particle (multiple by eachSize to scale it from grid coordinates to pixels)
            ctx.fillRect(col * eachSize, row * eachSize, eachSize, eachSize);
            // 👆 Put the following code here 👆
        }
    }
}
```
{: file="canvas.js" }
{: .nolineno }

This code iterates through each cell in our grid (which represents the canvas). For each cell, it gets the particle at that location. It then sets the `fillStyle` of the canvas context to the particle's color and draws a filled rectangle using `ctx.fillRect`. The `col * eachSize` and `row * eachSize` calculations convert the grid coordinates to pixel coordinates on the canvas. Once you have added the above code go ahead and run it and try to create some particles by clicking or dragging on the canvas. There is an error in the code and if you have your console open you will see errors there, will will fix them in the next section.

# Debugging errors

> **The code we just added has an error.** `grid` uses `null` to represent empty spaces. This will cause an error when we try to access `particle.color` because `particle` is `null`. Open your browser's developer tools (usually by pressing `F12` or right-clicking and selecting "Inspect"). Go to the "Console" tab. You should see many error messages.
{: .prompt-danger }

The error message you are seeing likely says something like `Cannot read properties of null (reading 'color')`. This means that at some point in our grid, there is a `null` value, and we are trying to access the color property of something that doesn't exist (i.e., `null`).

In our grid, we are using `null` to represent empty spaces where there is no particle. When the `redraw` function encounters a `null` value, it tries to access `particle.color`, which causes the error because `null` doesn't have a color property.

To fix this issue we need to add a `null` check to make sure we are not drawing any empty particles. This can be done by wrapping our drawing code in `if (particle != null)` or even easier `if (particle)`, this works because `null` is considered `false`, while our object (Class) is considered `true`.

```js
export function redraw() {
    // Clear previous frame
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    // Loop through all elements in the grid
    for (let row = 0; row < grid.length; row++) {
        for (let col = 0; col < grid[0].length; col++) {
            const particle = grid[row][col];

            // Check if there is a particle at (row, col). (null == false)
            if (particle) {
                // Get particle color
                ctx.fillStyle = particle.color;
                // Draw particle (multiple by eachSize to scale it from grid coordinates to pixels)
                ctx.fillRect(col * eachSize, row * eachSize, eachSize, eachSize);
            }
        }
    }
}
```
{: file="canvas.js" }
{: .nolineno }

> There are surprisingly few things in JavaScript that are considered `false`. Mainly `false`, `0`, `""` (empty string), `null` and `undefined`, and `NaN`. It is important to note that empty lists `[]` and objects `{}` are considered `true`.
{: .prompt-info }

**Try clicking on the canvas again. You should now see orange squares appearing where you click!**
