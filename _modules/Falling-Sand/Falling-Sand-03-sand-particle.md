---
title: "Sand Particle"
parent_post: Falling-Sand
module_number: 3
layout: module
media_subpath: /assets/tutorials/falling sand
---

Now that we can draw sand particles, let's make them fall!

# Inheritance

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

This is our base blueprint for all particles. It has a constructor that initializes the color and type properties, and it has two methods: `swap` and `update`.

Next, you'll see the `Sand` class:

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

Notice the `extends Particle` keyword. This tells JavaScript that the `Sand` class inherits from the `Particle` class. This means that the `Sand` class automatically gets the `color`, `type`, `swap`, and `update` properties and methods from the Particle class.

The `update(row, col)` method is where we'll define how a sand particle behaves over time. We need to modify this to move the sand down one row in the grid every time the `update` is called.

First we need a helper function to move a particle in the grid. Open the `canvas.js` file and find the `moveParticle` function. Replace the `TODO` comment with the following code:

```js
export function moveParticle(row, col, newRow, newCol, swap) {
    grid[newRow][newCol] = grid[row][col];
    return true;
}
```
{: file="canvas.js" }
{: .nolineno }

This function takes the current row and column `(row, col)` of a particle and the new row and column `(newRow, newCol)`, where we want to move it. It then simply copies the particle from its old position in the grid to its new position (ignore the `return true;` for now).

Now, open the `particles.js` file and find the `update` method inside the `Sand` class. Replace the `TODO` comment with the following code (ignore `this.swap` we will be using it later):

```js
update(row, col) {
    moveParticle(row, col, row+1, col, this.swap);
}
```
{: file="particles.js" }
{: .nolineno }


> Make sure you put this code in the `update` function of the `Sand` class **not** the base `Particle` class.
{: .prompt-warning }

This code calls the `moveParticle` function to move the sand particle from its current row and col to the row below it.

# Issues with moving

**There are two errors with the current implementation, they should become apparent as soon as you run this code and click on the screen. Lets fix them.**


When the sand reaches the bottom of the canvas, you might start seeing errors in the console again. This is because we are trying to move the sand to a row that doesn't exist (outside the bounds of our grid).

To fix this, we need to update the `checkBounds` function in `canvas.js` to checks if a given row and column are within the valid bounds of our grid.

> **TASK 1:** Modify the `checkBounds` function so it returns `true` if `(row, col)` is within the bounds of `grid` and `false` otherwise. After writing the function, use it in `moveParticle` to prevent particles from being moved out of bounds.
{: .prompt-warning }

<details>
<summary>Task 1: Hint</summary>
<blockquote>
Think about the dimensions of our grid. How can you check if a given row is within the valid range of rows? What about the column?
</blockquote>
</details>


> **Try to complete the task before moving on**
{: .prompt-danger }

**Answer (click to unblur):**

```js
export function checkBounds(row, col) {
    return row < grid.length && row >= 0 && col < grid[0].length && col >= 0;
}
```
{: file="canvas.js" }
{: .nolineno }
{: .blur }

`moveParticle` in `canvas.js` should look something like this:

```js
export function moveParticle(row, col, newRow, newCol, swap) {
    // 👇 Put the following code here 👇
    if (!checkBounds(row, col) || !checkBounds(newRow, newCol)) {
        return false;
    }
    // 👆 Put the following code here 👆


    // Rest of moveParticle
}
```
{: file="canvas.js" }
{: .nolineno }
{: .blur }

You might notice that when the sand moves down, it leaves a trail behind it. This is because we are only copying the sand particle to the new position and not removing it from its old position.

> **TASK 2:** Modify the `moveParticle` function in `canvas.js` to stop the particles from streaking as they fall.
{: .prompt-warning }

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

> **Try to complete the task before moving on**
{: .prompt-danger }

**Answer (click to unblur):**

```js
export function moveParticle(row, col, newRow, newCol, swap) {
    // Rest of moveParticle

    grid[newRow][newCol] = grid[row][col];
    grid[row][col] = null; // This line is the fix!
    return true;
}
```
{: file="canvas.js" }
{: .nolineno }
{: .blur }

Now, the errors when the sand hits the bottom and the streaking should be gone!

# Sand physics

Let's make our sand behave a bit more realistically. Currently, it just falls straight down. Lets add a check to make sure it doesn't overwrite other sand if there is already sand below it.


> **TASK 3:** Utilizing `getParticle` (which returns the particle at `(row, col)`), add a check in the `moveParticle` function in `canvas.js` to make sure a particle cannot move on top of another particle.
{: .prompt-warning }

> Remember if a gird location is empty it will contain the value `null`.
{: .prompt-info }

**Answer (click to unblur):**

```js
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
```
{: file="canvas.js" }
{: .nolineno }
{: .blur }

You might have noticed that we are returning `true` and `false` from the `moveParticle` function. This is to indicate wether a particle was moved or not, lets add some code in `particles.js` that moves the particle left if there is something below it.

Open the `particles.js` file and modify the `update` function in the `Sand` class. We'll use the return value of `moveParticle` to determine if the downward move was successful. If it was unable to move down, we'll try to move left.

```js
update(row, col) {
    // Try to move down
    if (!moveParticle(row, col, row+1, col, this.swap)) {
        // If cannot move down, try to move left
        moveParticle(row, col, row, col-1, this.swap);
    }
}
```
{: file="particles.js" }
{: .nolineno }


The sand looks better then before, but now once sand reaches the bottom it moves to the left until it hits the border of the canvas. Lets make sure that sand stops moving.

```js
update(row, col) {
    // Fall due to gravity
    let newRow = row + 1;

    // If nothing below move down
    if (!moveParticle(row, col, newRow, col, this.swap)) {
        moveParticle(row, col, newRow, col-1, this.swap);
    }
}
```
{: file="particles.js" }
{: .nolineno }

This works because when we try to move sand to the left, we are also trying to move it down, this results in the sand attempting to move below the screen which gets prevented by our bounds check. This also creates a satisfying pyramid shape.

Now lets add another check to have it move right if it can't move left.

```js
update(row, col) {
    // Fall due to gravity
    let newRow = row + 1;

    // If nothing below move down
    if (!moveParticle(row, col, newRow, col)) {
        // Try to move left
        if (!moveParticle(row, col, newRow, col-1, this.swap)) {
            moveParticle(row, col, newRow, col+1, this.swap)
        }
    }
}
```
{: file="particles.js" }
{: .nolineno }

> **CHALLENGE:** Mess around with the sand physics! What happens if you have the sand move two steps every update (`row+2` or `col+2`), or if you try to move left and right first?
{: .prompt-warning }

## Completion & Discussion Checklist

Before joining the group discussion or moving on to the next module, ensure you have completed the tasks, investigated the bugs, and are ready to discuss the questions below:

<details markdown="1">
<summary>Click to expand Completion & Discussion Checklist (7 Items)</summary>

| # | Type | Item | Prompt Preview |
| :-: | :--- | :--- | :--- |
| 1 | Bug Hunt | Empty Grid Null Check | Accessing `particle.color` on an empty grid cell causes a null error. Verify that a particle exists at `(row, col)` before reading its properties. |
| 2 | Question | Grid Bounds & Coordinates | Think about the dimensions of our grid. How can you check if a given row is within the valid range of rows? What about the column? |
| 3 | Question | Particle Reference vs. Copy | What do we use to represent an empty particle? Are we moving the particle or just making a new one? |
| 4 | Task | Implement `checkBounds()` | Modify the `checkBounds` function so it returns `true` if `(row, col)` is within the bounds of `grid` and `false` otherwise. Use it in `moveParticle` to prevent particles from moving out of bounds. |
| 5 | Task | Fix Particle Streaking | Modify the `moveParticle` function in `canvas.js` to stop the particles from streaking as they fall. |
| 6 | Task | Prevent Particle Overwrite | Utilizing `getParticle`, add a check in `moveParticle` in `canvas.js` to make sure a particle cannot move on top of another particle. |
| 7 | Challenge | Custom Physics Experimentation | Mess around with the sand physics! What happens if you have the sand move two steps every update (`row+2` or `col+2`), or if you try to move left and right first? |

</details>

Congratulations! You've completed the first part of the Falling Sand tutorial. You can now create and make sand particles fall and react to simple physics. In the next part, we'll introduce more particle types and make them interact with each other.
