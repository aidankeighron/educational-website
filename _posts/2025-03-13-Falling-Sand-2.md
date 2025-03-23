---
title: Falling Sand 1
author: aidan
date: 2025-03-20 12:00:00 +0800
categories: [JavaScript, FallingSand]
tags: [JavaScript, Easy]
description: In this project, you will learn the fundamentals of programming by building a fun and interactive simulation of different particles.
comments: false
pin: true
---

## About the project

Welcome to the Falling Sand tutorial! In this project, you will learn the fundamentals of programming by building a fun and interactive simulation of different particles.

**What you will learn:**

- Basic JavaScript concepts like variables, the Document Object Model (DOM), and event listeners.
- How to use the HTML Canvas element to draw graphics on a webpage.
- Fundamental game logic and physics simulation.
- Object-oriented programming concepts like classes and inheritance.
- Problem-solving and debugging skills.

**What you will make:**

By the end of this tutorial, you will have a working simulation where you can click on the screen to create different types of particles that will fall and interact based on simple rules.

**Further possibilities:**

Once you've completed this tutorial, you can expand on it in many ways! You could add more particle types with unique behaviors (like fire that burns wood, or ice that melts), implement more complex physics like animals and plants. The possibilities are endless!

## Setup

First, you'll need to get the project code and set up your development environment.

1. **Set up your IDE:** We will be using VS Code in this tutorial but you can use any IDE, you can download VS Code from [https://code.visualstudio.com/](https://code.visualstudio.com/){:target="\_blank"}. Install the Live Server extension in Visual Studio Code: open the Extensions marketplace (usually by clicking on the four squares icon on the left sidebar) and search for "Live Server" by Ritwick Dey. Click "Install".

2. **Download the starter code:** If you don't have it already, download GitHub desktop: [https://desktop.github.com/download/](https://desktop.github.com/download/){:target="\_blank"}. Fork the project on GitHub: Go to [https://github.com/aidankeighron/Falling-Sand-Tutorial](https://github.com/aidankeighron/Falling-Sand-Tutorial){:target="\_blank"} and click the "Fork" button in the top right corner. This will create a copy of the project in your own GitHub account. Then open your newly created fork and clone it in GitHub desktop.

## Running the project

Now let's get the project running in your browser.

- **Open the forked project in VS Code:** Go to "File" -> "Open Folder" and select the directory where you cloned your forked repository.
- **Start Live Server:** Open the `index.html` file in VS Code. Right-click anywhere in the file and select "Open with Live Server".
- **Open in a new tab (if it didn't automatically):** Live Server will usually open the webpage in your default browser. If it doesn't, you should see a message in the VS Code status bar at the bottom indicating the port number (e.g., "Port: 5500"). Open a new tab in your browser and navigate to http://127.0.0.1:5500/ (or the port number shown in VS Code).
- 
You should now see a webpage with the title "Falling Sand" and a blank rectangle (the canvas) in the center.

## Basic JavaScript knowledge

This section will cover some fundamental JavaScript concepts that we'll be using in this tutorial. If you are already comfortable with JavaScript, feel free to skip to the next section.

### HTML DOM (accessing HTML DOM using `getElementById`)

The HTML Document Object Model (DOM) represents the structure of your HTML document as a tree of objects. JavaScript can interact with this tree to dynamically change the content and behavior of your webpage.

`getElementById()` is a JavaScript method that allows you to access a specific HTML element by its id attribute. In our `index.html` file, you'll find elements with IDs like canvas, speedRange, and clear-button. We can access these elements in our JavaScript code like this:

```js
const canvasElement = document.getElementById('canvas');
const speedSliderElement = document.getElementById('speedRange');
const clearButtonElement = document.getElementById('clear-button');

console.log(canvasElement); // This will log the canvas HTML element to the console.
```

### Event listeners

Event listeners allow you to respond to specific events that happen on your webpage, such as a user clicking a button or moving their mouse. We attach event listeners to HTML elements using JavaScript.

Here's an example of how we can add an event listener to our "Clear Screen" button:

```js
const clearButtonElement = document.getElementById('clear-button');

clearButtonElement.addEventListener('click', function() {
  console.log('Clear button clicked!');
});
```

In this code:

- We get the button element using its ID.
- We use the `addEventListener()` method to attach a function to the `'click'` event.
- The function inside `addEventListener` will be executed every time the button is clicked.


## Draw particles on click

Now let's start drawing particles on the canvas when you click the mouse.

### Brief overview of how canvas works

The HTML `<canvas>` element is used to draw graphics on a webpage using JavaScript. It's like a blank painting surface that you can control with code. To draw on the canvas, you first need to get its 2D rendering context. This context provides methods for drawing shapes, text, images, and more.

In our `canvas.js` file, you'll see this code at the top:

```js
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext("2d");
```
{: file="canvas.js" }
{: .nolineno }

This code gets the canvas element from our HTML and then gets its 2D rendering context, which we store in the `ctx` variable. We'll use this `ctx` variable to draw our particles.

### Mouse Listeners

In the `canvas.js` file, you'll also find a function called `setUpMouseListeners()`:

```js
export function setUpMouseListeners() {
    canvas.addEventListener("mousedown", (event) => {
        isDragging = true;
        mousePosition = {clientX: event.clientX, clientY: event.clientY};
    });
    canvas.addEventListener("mousemove", (event) => {
        mousePosition = event;
    });
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

In our `main.js` file, inside the `update()` function, you'll see how we use these listeners to create particles when the mouse is clicked and dragged:

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

### Draw particles on the canvas

Now, let's write the code that actually draws the particles on the canvas. Open the `canvas.js` file and find the `redraw()` function. You'll see a `TODO` comment inside the inner for loop.

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

This code iterates through each cell in our grid (which represents the canvas). For each cell, it gets the particle at that location. It then sets the `fillStyle` of the canvas context to the particle's color and draws a filled rectangle using `ctx.fillRect()`. The `col * eachSize` and `row * eachSize` calculations convert the grid coordinates to pixel coordinates on the canvas. Once you have added the above code go ahead and run it and try to create some particles by clicking or dragging on the canvas.

### Debugging errors

> **The code we just added has an error.** `grid` uses `null` to represent empty spaces. This will cause an error when we try to access `particle.color` because `particle` is `null`. Open your browser's developer tools (usually by pressing `F12` or right-clicking and selecting "Inspect"). Go to the "Console" tab. You should see many error messages.
{: .prompt-danger }

The error message you are seeing likely says something like `Cannot read properties of null (reading 'color')`. This means that at some point in our grid, there is a `null` value, and we are trying to access the color property of something that doesn't exist (i.e., `null`).

In our grid, we are using `null` to represent empty spaces where there is no particle. When the `redraw()` function encounters a `null` value, it tries to access `particle.color`, which causes the error because `null` doesn't have a color property.

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

## Sand particle

Now that we can draw sand particles, let's make them fall!

### Inheritance

In programming, `classes` are like blueprints for creating `objects`. An `object` is a collection of data (properties) and actions (methods). In our project, we'll use classes to represent different types of particles, like `Sand`.

If you are confused about what classes are and how they work check out [this](https://www.w3schools.com/js/js_classes.asp){:target="\_blank"} breakdown

Inheritance is a powerful concept in object-oriented programming where a new class (called a subclass or derived class) can inherit properties and methods from an existing class (called a superclass or base class). This helps us write more organized and reusable code.

In our `particles.js` file, you'll see a base Particle class:

```js
/**
 * Base particle class
 */
class Particle {
    constructor() {
        this.color = "";
        this.type = "";
    }

    /**
     * Returns true if the particle should swap with other when trying
     * to move onto the same grid location as {@link other}.
     *
     * EX: Let sand sink below water
     *
     * @param {Particle} other
     * @returns {boolean} Should the particle swap
     */
    swap(other) {
        return false;
    }

    /**
     * Update the particle at location (row, col)
     *
     * @param {number} row
     * @param {number} col
     */
    update(row, col) {

    }
}
```
{: file="particles.js" }
{: .nolineno }

This is our base blueprint for all particles. It has a constructor that initializes the color and type properties, and it has two methods: `swap()` and `update()`.

Next, you'll see the Sand class:

```js
/**
 * Sand particle
 */
export class Sand extends Particle {
    constructor() {
        super(); // Calls the constructor of the parent class (Particle)
        this.color = "orange";
        this.type = "sand";
    }

    swap(other) {
        // TODO make sand fall under the water
    }

    update(row, col) {
        // TODO update sand
    }
}
```
{: file="particles.js" }
{: .nolineno }

Notice the `extends Particle` keyword. This tells JavaScript that the `Sand` class inherits from the `Particle` class. This means that the `Sand` class automatically gets the `color`, `type`, `swap()`, and `update()` properties and methods from the Particle class.

The `update(row, col)` method is where we'll define how a sand particle behaves over time. We need to modify this to move the sand down one row in the grid every time the `update()` is called.

First we need a helper function to move a particle in the grid. Open the `canvas.js` file and find the `moveParticle()` function. Replace the `TODO` comment with the following code:

```js
export function moveParticle(row, col, newRow, newCol, swap) {
    grid[newRow][newCol] = grid[row][col];
    return true;
}
```
{: file="canvas.js" }
{: .nolineno }

This function takes the current row and column `(row, col)` of a particle and the new row and column `(newRow, newCol)`, where we want to move it. It then simply copies the particle from its old position in the grid to its new position (ignore the `return true;` for now).

Now, open the `particles.js` file and find the `update()` method inside the `Sand` class. Replace the `TODO` comment with the following code:

```js
update(row, col) {
    moveParticle(row, col, row+1, col);
}
```
{: file="particles.js" }
{: .nolineno }


> Make sure you put this code in the `update` function of the `Sand` class **not** the base `Particle` class.
{: .prompt-warning }

This code calls the `moveParticle()` function to move the sand particle from its current row and col to the row below it.

### Issues with moving

**There are two errors with the current implementation, they should become apparent as soon as you run this code and click on the screen. Lets fix them.**


When the sand reaches the bottom of the canvas, you might start seeing errors in the console again. This is because we are trying to move the sand to a row that doesn't exist (outside the bounds of our grid).

To fix this, we need to update the `checkBounds` function to checks if a given row and column are within the valid bounds of our grid.

> Task 1: Modify the `checkBounds` functions so it returns `true` if the `(row, col)` is within the bounds of `grid` and `false` otherwise. After writing function, use it in `moveParticle` to prevent a particle from being moved out of bounds. 
{: .prompt-tip }

<details>
<summary>Task 1: Hint</summary>
<blockquote>
Think about the dimensions of our grid. How can you check if a given row is within the valid range of rows? What about the column?
</blockquote>
</details>

<br>

> **Try to complete the task before moving on**
{: .prompt-danger }

**Answer:**

```js
export function checkBounds(row, col) {
    return row < grid.length && row >= 0 && col < grid[0].length && col >= 0;
}
```
{: file="canvas.js" }
{: .nolineno }

`moveParticle` should look something like this:

```js
export function moveParticle(row, col, newRow, newCol, swap) {
    if (!checkBounds(row, col) || !checkBounds(newRow, newCol)) {
        return false;
    }

    grid[newRow][newCol] = grid[row][col];
    grid[row][col] = null;
    return true;
}
```
{: file="canvas.js" }
{: .nolineno }

You might notice that when the sand moves down, it leaves a trail behind it. This is because we are only copying the sand particle to the new position and not removing it from its old position.

> Task 2: Modify the `moveParticle` function to stop the particles streaking as they fall
{: .prompt-tip }

<details>
<summary>Task 2: Hint</summary>
<blockquote>
What do we use to represent an empty particle

<details>
<summary>Hint 2</summary>
<blockquote>
Are we moving the particle or just making a new one
</blockquote>
</details>

</blockquote>
</details>

<br>

> **Try to complete the task before moving on**
{: .prompt-danger }

**Answer:**

```js
export function moveParticle(row, col, newRow, newCol, swap) {
    grid[newRow][newCol] = grid[row][col];
    grid[row][col] = null; // This line is the fix!
    return true;
}
```
{: file="canvas.js" }
{: .nolineno }

Now, the errors when the sand hits the bottom and the streaking should be gone!

### Sand physics

Let's make our sand behave a bit more realistically. Currently, it just falls straight down. We should add a check to make sure it doesn't overwrite other sand if there is already sand below it.

Add a check in moveParticle to make sure sand does not overwrite another sand if it is below:

Open the canvas.js file and modify the moveParticle() function again. Before actually moving the particle, we should check if there is already a particle at the newRow and newCol position. If there is, and it's not a particle we can swap with (we'll get to swapping later), then we shouldn't move the current particle.

JavaScript

export function moveParticle(row, col, newRow, newCol, swap) {
    if (!checkBounds(row, col) || !checkBounds(newRow, newCol)) {
        return false;
    }

    // 👇 Add this check 👇
    if (getParticle(newRow, newCol)) {
        return false;
    }
    // 👆 Add this check 👆

    grid[newRow][newCol] = grid[row][col];
    grid[row][col] = null;
    return true;
}
We've added a call to getParticle(newRow, newCol). This function (which is already defined in canvas.js) returns the particle at the specified coordinates, or null if there is no particle. If getParticle() returns a truthy value (meaning there is a particle), we return false from moveParticle(), preventing the current particle from moving.

In the update function use the return value of moveParticle to move the sand left if there is something below it. Just left not left down.

Now, let's make the sand try to move to the left if it can't move straight down. Open the particles.js file and modify the update() function in the Sand class. We'll use the return value of moveParticle() to determine if the downward move was successful. If it wasn't, we'll try to move left.

JavaScript

update(row, col) {
    // Try to move down
    if (!moveParticle(row, col, row+1, col)) {
        // If cannot move down, try to move left
        moveParticle(row, col, row, col-1);
    }
}
We first try to move the sand down. The ! in front of moveParticle() checks if the function returned false (meaning the move was unsuccessful). If it was unsuccessful, we then try to move the sand one column to the left (col - 1) in the same row.

Have them try and fix the issue of sand always moving the to right.

You might notice that the sand now always tries to move to the left. This isn't quite right. We want it to move to the left or the right randomly if it can't move straight down.

Write some hints:

<details>
<summary>Hint 1</summary>
We need to introduce some randomness into our decision of whether to move left or right.
</details>

<details>
<summary>Hint 2</summary>
Look at the util.js file. There's a function there that might be useful for generating random numbers.
</details>

<details>
<summary>Hint 3</summary>
Use the getRandomInt() function to generate either -1 (for left) or 1 (for right) and add that to the col value when calling moveParticle().
</details>

answer:

JavaScript

// Fall due to gravity
let newRow = row + 1;
// Make sure sand does not fall off screen
if (!checkBounds(newRow, col)) {
    return;
}

if (!moveParticle(row, col, newRow, col)) {
    const direction = Math.random() < 0.5 ? -1 : 1; // Randomly choose -1 or 1
    moveParticle(row, col, row, col + direction);
}
The row+1 (newRow) for both move operations is important because without it it starts to behave like water.

Explain that and have them do this (temporarily):

If you change the second moveParticle call to use row + 1 instead of row, you'll see the sand start to spread out more like water. This is because it's trying to move diagonally down-left or down-right instead of just left or right.

Have them do this (temporarily):

JavaScript

// Fall due to gravity
let newRow = row + 1;
// Make sure sand does not fall off screen
if (!checkBounds(newRow, col)) {
    return;
}

if (!moveParticle(row, col, newRow, col)) {
    const direction = Math.random() < 0.5 ? -1 : 1; // Randomly choose -1 or 1
    moveParticle(row, col, row + 1, col + direction); // < this (from row to row + 1) [row to newRow] it starts to look like water
}
OPTIONAL Have them mess around with change newRow and newCol, (or even row and col) you can do some pretty cool stuff.

Feel free to experiment with changing the newRow and newCol values in the moveParticle() calls. You can try having the sand move up, diagonally, or even skip rows or columns. See what interesting behaviors you can create!

Remind them to change it back (maybe give them the entire update function? just to be sure they are on the same track):

Make sure to change the update() function back to the correct behavior for sand:

JavaScript

update(row, col) {
    // Fall due to gravity
    let newRow = row + 1;
    // Make sure sand does not fall off screen
    if (!checkBounds(newRow, col)) {
        return;
    }

    if (!moveParticle(row, col, newRow, col)) {
        const direction = Math.random() < 0.5 ? -1 : 1; // Randomly choose -1 or 1
        moveParticle(row, col, row, col + direction);
    }
}
Add another check for if it can't move right have it move left.

Currently, the sand tries to move left or right with equal probability. Let's refine this so that if it tries to move in one horizontal direction and can't, it tries the other direction.

JavaScript

update(row, col) {
    // Fall due to gravity
    let newRow = row + 1;
    // Make sure sand does not fall off screen
    if (!checkBounds(newRow, col)) {
        return;
    }

    if (!moveParticle(row, col, newRow, col)) {
        let moved = false;
        if (Math.random() < 0.5) {
            moved = moveParticle(row, col, row, col + 1); // Try right
            if (!moved) {
                moveParticle(row, col, row, col - 1); // Try left if right failed
            }
        } else {
            moved = moveParticle(row, col, row, col - 1); // Try left
            if (!moved) {
                moveParticle(row, col, row, col + 1); // Try right if left failed
            }
        }
    }
}
row+1 or newRow works just make sure they are constant.

Remember that when checking for movement downwards, you should consistently use either row + 1 or the newRow variable that you've already declared. Mixing them up can lead to unexpected behavior.

Try to think of a challenge they can do, or an optional task, or just messing around with the system.

Challenge: Try to make the sand settle more towards the center. You could achieve this by slightly increasing the probability of it trying to move towards the center column if it can't move straight down.

END OF TUTORIAL 1

Congratulations! You've completed the first part of the Falling Sand tutorial. You can now create and make sand particles fall and react to simple physics. In the next part, we'll introduce more particle types and make them interact with each other.

## Water particle
Now let's add a new particle type: Water!

Explain water particle code from solution branch:

You can find the code for the Water particle in the solution branch of the repository: https://github.com/aidankeighron/Falling-Sand-Tutorial/tree/solution. Open the particles.js file in that branch and take a look at the Water class.

Ignore the convert from dirt to grass for now; we'll get to that later. Focus on the basic movement of the water particle. You'll notice that the update() function for water is a bit different from sand.

Explain how the random stuff works:

JavaScript

if (getRandomInt(0, 4)) { // body }
This code snippet uses the getRandomInt() function from util.js to generate a random integer between 0 (inclusive) and 4 (inclusive). The if condition will evaluate to true if the result of getRandomInt() is any number other than 0. In JavaScript, 0 is considered a "falsy" value, while any non-zero number is "truthy". Therefore, the body of this if statement has a 75% chance (since there are 4 non-zero numbers out of 5 possibilities) of being executed.

Explain how they can invert the probability with !

You can easily invert this probability using the ! (NOT) operator:

JavaScript

if (!getRandomInt(0, 4)) { // body }
Now, the if condition will only be true if getRandomInt(0, 4) returns 0. This means the body of the if statement will now have a 25% chance of being executed.

Task (don't give them answer and all) Have them mess around with water physics (what random range to use), extra movement options, floating water? random change to move to a random location, have them get creative with it (add what I just mentioned as examples). You need to add x new options?

Task: Experiment with the Water particle's update() function in particles.js. Try the following:

Change the random range: Modify the arguments to getRandomInt() to see how it affects the probability of water moving horizontally. What happens if you use getRandomInt(0, 2)? What about getRandomInt(0, 10)?

<details>
<summary>Hint</summary>
Look for the lines in the Water's update() function that use getRandomInt(). The second argument controls the upper limit of the random number (exclusive).
</details>

Add more movement options: Currently, water tries to move down, then left or right. Try adding a small chance for it to move diagonally down-left or down-right.

<details>
<summary>Hint</summary>
You can add more if conditions within the update() function, using getRandomInt() to control the probability of these new movements. Remember to use moveParticle() to actually move the water.
</details>

Floating water? Try making water have a small chance to move upwards. What parameters for getRandomInt() would you use for a very low probability?

<details>
<summary>Hint</summary>
Moving upwards would involve decreasing the row value in moveParticle(). Think about how to make this happen very rarely.
</details>

Random change to move to a random location: This is a more advanced challenge! Try giving water a very small chance to teleport to a completely random empty location on the grid.

<details>
<summary>Hint</summary>
You'll need to use the getRandomLocation() function from canvas.js to get a random row and col. Make sure the target location is empty before moving the water there!
</details>

Have fun experimenting and see what kind of interesting water behaviors you can create! Remember to refresh your browser after making changes to the code.

## Swap function
Now let's implement the swap() function in our particles.js file. This will allow us to define how different particles interact when they try to move into the same space.

Explain the swap functions and have them implement a system where sand falls beneath water.

Currently, if sand tries to move into a space occupied by water, it will just stop. We want to make it so that sand falls beneath water. To do this, we need to implement the swap() method in both the Sand and Water classes.

Open the particles.js file.

Task: Implement the swap() method in the Sand class so that it returns true if the other particle is an instance of the Water class. Also, implement the swap() method in the Water class so that it returns true if the other particle is an instance of the Sand class.

<details>
<summary>Hint for Sand's swap()</summary>
You can use the instanceof operator in JavaScript to check if an object is an instance of a particular class. For example, other instanceof Water will return true if other is a Water object.
</details>

<details>
<summary>Hint for Water's swap()</summary>
Similarly, in the Water class's swap() method, you'll check if other is an instance of the Sand class.
</details>

<details>
<summary>Answer for Sand's swap()</summary>

JavaScript

swap(other) {
    return other instanceof Water;
}
</details>

<details>
<summary>Answer for Water's swap()</summary>

JavaScript

swap(other) {
    return other instanceof Sand;
}
</details>

Now, open the canvas.js file and modify the moveParticle() function to use the swap parameter.

Task: Modify the moveParticle() function to check if there is a particle at the newRow and newCol. If there is, and the swap parameter is a function, call the swap function of the particle at the current location (grid[row][col]) with the particle at the new location as an argument. If the swap function returns true, then swap the positions of the two particles in the grid.

JavaScript

export function moveParticle(row, col, newRow, newCol, swap) {
    if (!checkBounds(row, col) || !checkBounds(newRow, newCol)) {
        return false;
    }

    const targetParticle = getParticle(newRow, newCol);
    const currentParticle = grid[row][col];

    if (targetParticle) {
        // 👇 Add this check and swap logic 👇
        if (swap && currentParticle.swap(targetParticle)) {
            grid[newRow][newCol] = currentParticle;
            grid[row][col] = targetParticle;
            return true;
        } else {
            return false;
        }
        // 👆 Add this check and swap logic 👆
    }

    grid[newRow][newCol] = grid[row][col];
    grid[row][col] = null;
    return true;
}
Now, open the particles.js file again and update the Sand and Water update() functions to pass a reference to their own swap method to the moveParticle() function.

Task: Modify the update() function for both Sand and Water to pass this.swap as the fifth argument to the moveParticle() function.

<details>
<summary>Answer for Sand's update()</summary>

JavaScript

update(row, col) {
    let newRow = row + 1;
    if (!checkBounds(newRow, col)) {
        return;
    }

    if (!moveParticle(row, col, newRow, col, this.swap)) {
        const direction = Math.random() < 0.5 ? -1 : 1;
        moveParticle(row, col, row, col + direction, this.swap);
    }
}
</details>

<details>
<summary>Answer for Water's update() (simplified for basic movement)</summary>

JavaScript

update(row, col) {
    // Try to move down
    let newRow = row + 1;
    if (checkBounds(newRow, col) && !getParticle(newRow, col)) {
        moveParticle(row, col, newRow, col, this.swap);
        return;
    }

    // Try to move left or right randomly
    const direction = Math.random() < 0.5 ? -1 : 1;
    const newCol = col + direction;
    if (checkBounds(row, newCol) && !getParticle(row, newCol)) {
        moveParticle(row, col, row, newCol, this.swap);
    }
}
</details>

Now, when you run the simulation and create both sand and water, you should see the sand fall beneath the water!

solution: https://github.com/aidankeighron/Falling-Sand-Tutorial/tree/solution

You can refer to the solution branch on GitHub if you encounter any issues.

## Add more particles (Rock and Dirt)

Let's add two more particle types to our simulation: Rock and Dirt.

Rock is just an extension of the base particle class with a different color and name and nothing else (you don't even need to define update or swap functions because it inherits it from Particle). DO NOT have them just create empty functions, explain the inheritance.

Open the particles.js file.

Task: Create a new class called Rock that extends the Particle class. In its constructor, set the color to "gray" and the type to "rock". You don't need to add any update() or swap() methods to the Rock class.

JavaScript

/**
 * Rock particle
 */
export class Rock extends Particle {
    constructor() {
        super(); // Call the constructor of the Particle class
        this.color = "gray";
        this.type = "rock";
    }
    // No update or swap needed!
}
Because the Rock class extends the Particle class, it automatically inherits the color and type properties from the Particle constructor. We just override the default values in the Rock constructor. Since rock shouldn't move or interact with other particles in this basic implementation, we don't need to define the update() or swap() methods. It will simply stay where it is placed.

Dirt inherits from Sand, you just need to change the color and title, the inheritance will copy the update and swap function (explain this).

Now let's add the Dirt particle. We want dirt to behave just like sand, but with a different color.

Task: Create a new class called Dirt that extends the Sand class. In its constructor, set the color to "brown" and the type to "dirt". You don't need to add any update() or swap() methods to the Dirt class.

JavaScript

/**
 * Dirt particle
 */
export class Dirt extends Sand {
    constructor() {
        super(); // Call the constructor of the Sand class
        this.color = "brown";
        this.type = "dirt";
    }
    // No update or swap needed! It inherits from Sand.
}
Since the Dirt class extends the Sand class, it inherits all the properties and methods of Sand, including its constructor, update(), and swap() methods. We only need to override the color and type in the Dirt constructor to give it its unique appearance.

Finally, we need to add these new particle types to the dropdown menu in our index.html file and update the checkParticleType() function in particles.js to handle them.

Task:

Open the index.html file and add <option> tags for "Stone" and "Dirt" to the <select> element with the id="particle". Make sure the value attribute matches the type you set in the Rock and Dirt classes (e.g., "Stone" and "Dirt").

HTML

<select name="particle" id="particle">
    <option value="Sand">Sand</option>
    <option value="Water">Water</option>
    <option value="Stone">Stone</option> <!- Add this -->
    <option value="Dirt">Dirt</option>   <!- Add this -->
    </select>
Open the particles.js file and update the checkParticleType() function to create and return instances of the Rock and Dirt classes when their corresponding values are selected in the dropdown.

JavaScript

export function checkParticleType(value) {
    if (value == "Sand") {
        return new Sand();
    } else if (value == "Water") {
        return new Water();
    } else if (value == "Stone") { // Add this
        return new Rock();
    } else if (value == "Dirt") {   // Add this
        return new Dirt();
    }
    return null;
}
Now you should be able to select Rock and Dirt from the dropdown and create them in your simulation! Rock will stay in place, and Dirt will fall like sand.

## Have water convert dirt into grass

Let's add an interesting interaction: when water touches dirt, it will turn the dirt into grass!

Create grass it inherits from sand, don't give them the code for this.

Task: Create a new class called Grass in particles.js that extends the Sand class. In its constructor, set the color to "lightgreen" and the type to "grass". Also, add an <option> tag for "Grass" in the index.html file and update the checkParticleType() function in particles.js.

JavaScript

/**
 * Grass particle
 */
export class Grass extends Sand {
    constructor() {
        super();
        this.color = "lightgreen";
        this.type = "grass";
    }
}
HTML

<select name="particle" id="particle">
    <option value="Sand">Sand</option>
    <option value="Water">Water</option>
    <option value="Stone">Stone</option>
    <option value="Dirt">Dirt</option>
    <option value="Grass">Grass</option> <!- Add this -->
    </select>
JavaScript

export function checkParticleType(value) {
    if (value == "Sand") {
        return new Sand();
    } else if (value == "Water") {
        return new Water();
    } else if (value == "Stone") {
        return new Rock();
    } else if (value == "Dirt") {
        return new Dirt();
    } else if (value == "Grass") { // Add this
        return new Grass();
    }
    return null;
}
In the update function for water before anything else happens check if the particle below is dirt and if it is, remove the water and replace the dirt with grass:

Now, let's implement the logic for water turning dirt into grass. Open the particles.js file and find the update() function in the Water class.

JavaScript

update(row, col) {
    // Make water turn dirt into grass when it touches it
    if (getParticle(row+1, col)?.type == "dirt") {
        // Remove water and change dirt to grass
        setParticle(row+1, col, new Grass());
        setParticle(row, col, null);
        return;
    }

    // Try to move down
    let newRow = row + 1;
    if (checkBounds(newRow, col) && !getParticle(newRow, col)) {
        moveParticle(row, col, newRow, col, this.swap);
        return;
    }

    // Try to move left or right randomly
    const direction = Math.random() < 0.5 ? -1 : 1;
    const newCol = col + direction;
    if (checkBounds(row, newCol) && !getParticle(row, newCol)) {
        moveParticle(row, col, row, newCol, this.swap);
    }
}
At the very beginning of the update() function for Water, we now check if the particle directly below the water (row + 1, col) exists and if its type is "dirt". We use the optional chaining operator (?.) to safely access the type property even if getParticle(row + 1, col) returns null.

If the particle below is indeed dirt, we use the setParticle() function (from canvas.js) to replace the dirt with a new Grass particle. We also remove the water particle by setting the current cell (row, col) to null. The return statement here ensures that the water doesn't try to move further after converting the dirt to grass.

Now, try creating some dirt and then some water on top of it – you should see the dirt turn into grass!

## Expand

Add x new particles.

Here are a couple of ideas for new particles you can add to your simulation:

Some suggestions and how they work:

Fire:

In the Particle class add a property called duration, and one called maxDuration, every time update is called increase duration, maxDuration should be initialized in the constructor using a random number between x, and y. Fire works like water but moving up instead of down. And after the duration has ended it is removed. Show code examples of duration system?

JavaScript

class Particle {
    constructor() {
        this.color = "";
        this.type = "";
        this.duration = 0; // Current duration
        this.maxDuration = 0; // Maximum duration
    }

    swap(other) {
        return false;
    }

    update(row, col) {
        this.duration++;
        if (this.duration > this.maxDuration) {
            setParticle(row, col, null); // Remove particle if duration exceeded
            return;
        }
    }
}

export class Fire extends Particle {
    constructor() {
        super();
        this.color = "red";
        this.type = "fire";
        this.maxDuration = getRandomInt(20, 50); // Set random lifespan
    }

    update(row, col) {
        super.update(row, col); // Call the base Particle's update method

        // Try to move up
        if (checkBounds(row - 1, col) && !getParticle(row - 1, col)) {
            moveParticle(row, col, row - 1, col, this.swap);
            return;
        }

        // Try to move left or right randomly
        const direction = Math.random() < 0.5 ? -1 : 1;
        const newCol = col + direction;
        if (checkBounds(row, newCol) && !getParticle(row, newCol)) {
            moveParticle(row, col, row, newCol, this.swap);
        }
    }
}
Remember to add "Fire" to the dropdown in index.html and handle it in checkParticleType() in particles.js.

Steam:

Moves like water but upwards, has a very small chance to disappear, if it is at the top of the screen it has a change to turn into water.

JavaScript

export class Steam extends Particle {
    constructor() {
        super();
        this.color = "lightgray";
        this.type = "steam";
    }

    update(row, col) {
        // Small chance to disappear
        if (Math.random() < 0.05) {
            setParticle(row, col, null);
            return;
        }

        // Try to move up
        if (checkBounds(row - 1, col) && !getParticle(row - 1, col)) {
            moveParticle(row, col, row - 1, col, this.swap);
            return;
        }

        // Try to move left or right randomly
        const direction = Math.random() < 0.5 ? -1 : 1;
        const newCol = col + direction;
        if (checkBounds(row, newCol) && !getParticle(row, newCol)) {
            moveParticle(row, col, row, newCol, this.swap);
            return;
        }

        // Chance to turn into water at the top
        if (row === 0 && Math.random() < 0.1) {
            setParticle(row, col, new Water());
        }
    }
}
Again, remember to add "Steam" to the dropdown and checkParticleType().

Many more
Think about other real-world substances or fantastical elements and how they might behave. Could you add:

Wood: A solid particle that doesn't move.
Acid: A particle that falls and dissolves other particles it touches.
Ice: A particle that falls and can freeze water.
Lava: A particle that falls and can burn other particles.
The possibilities are endless! Get creative and have fun experimenting!