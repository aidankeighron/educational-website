---
title: Falling Sand 1
author: aidan
date: 2025-03-25 12:00:00 +0800
categories: [JavaScript]
tags: [JavaScript, Easy]
---

# About the project

Description of what the project is, what they will learn, and what they will make at the end. Also talk about how you can do further after the project is done.

# Setup

Install the codebase from GitHub (fork project): https://github.com/aidankeighron/Falling-Sand-Tutorial, install live server extension https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer

*Later* full setup of vs code

# Running the project

Startup live server and open the new webpage in a new tab

# Basic JavaScript knowledge

Feel free to skip this topic if you are already comfortable with JavaScript

let vs const

HTML Dom (accessing HTML Dom using getElementByID)

Event listeners

Try to add pictures or code examples. You can add a link to an external website but ONLY for extra reading, everything the should need should be within this website 

# Draw particles on click

Brief overview of how canvas works, explain mouse listeners (in context of code)

Write body of redraw function in canvas.js:

Have them put this inside the body of the for loop first (you make want to show the for loop too or the whole method so they know exactly where it goes)

```js
const particle = grid[row][col];

// Get particle color
ctx.fillStyle = particle.color;
// Draw particle (multiple by eachSize to scale it from grid coordinates to pixels)
ctx.fillRect(col * eachSize, row * eachSize, eachSize, eachSize);
```

This ^ code has an error, grid uses null to represent empty spaces so this will cause an error, teach them how to open their inspector and see the error.
Explain briefly what the error message means, and explain how we use null to represent empty spaces.

Then give them this working code or have them add a null check themselves (explain how null is false):

```js
const particle = grid[row][col];

// Check if there is a particle at (row, col). (null == false)
if (particle) {
    // Get particle color
    ctx.fillStyle = particle.color;
    // Draw particle (multiple by eachSize to scale it from grid coordinates to pixels)
    ctx.fillRect(col * eachSize, row * eachSize, eachSize, eachSize);
}
```

# Move sand particle

Explain JS classes and go over inheritance. Explain the particle class and the update function. Have them move the sand down one row every time update is called.

Give them this code for moveParticle

```js
grid[newRow][newCol] = grid[row][col];
```

And this for the sand particle update function

```js
moveParticle(row, col, row+1, col);
```

Have them try and fix the streaking issue, (hint its in the moveParticle function) (hint 2 what do we use to represent and empty particle)
(hint 3 are we currently moving the particle or just making a new one)

Answer

moveParticle
```js
grid[row][col] = null;
```

The will get errors when sand hits the bottom. Have them create the checkBounds function to return false if the sand is outside the grid.

Don't give them the answer (this will probably be a 335 nested dropdown where they drop down through a few hints before getting the answer)

```js
row < grid.length && row >= 0 && col < grid[0].length && col >= 0;
```

Have them use checkBounds to check the bounds of (row, col) and (newRow, newCol) and return false if either of the checks fail

Move particle should look something like this

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

# Sand physics

Add a check in move particle to make sure sand does not overwrite another sand if it is below

```js
if (getParticle(newRow, newCol)) {
    return false;
}
```

In the update function use the return value of moveParticle to move the sand left if there is something below it. Just left not left down

```js
if (!moveParticle(row, col, row+1, col)) {
    moveParticle(row, col, row, col+1)
}
```

Have them try and fix the issue of sand always moving the to right

Write some hints

answer:

```js
// Fall due to gravity
let newRow = row + 1;
// Make sure sand does not fall off screen
if (!checkBounds(newRow, col)) {
    return;
}

if (!moveParticle(row, col, newRow, col)) {
    moveParticle(row, col, newRow, col+1)
}
```

The row+1 (newRow) for both move operations is important because without it it starts to behave like water

explain that and have them do this (temporarily)

```js
// Fall due to gravity
let newRow = row + 1;
// Make sure sand does not fall off screen
if (!checkBounds(newRow, col)) {
    return;
}

if (!moveParticle(row, col, newRow, col)) {
    moveParticle(row, col, row, col+1) // < this (from newRow to row) [row+1 to row] it starts to look like water
}
```

OPTIONAL Have them mess around with change newRow and newCol, (or even row and col) you can do some pretty cool stuff. 

Remind them to change it back (maybe give them the entire update function? just to be sure they are on the same track)

Add another check for if it can't move right have it move left

```js
if (!moveParticle(row, col, row+1, col)) {
    if (!moveParticle(row, col, row+1, col+1)) {
        moveParticle(row, col, row+1, col-1)
    }
}
```

row+1 or newRow works just make sure they are constant

Try to think of a challenge they can do, or an optional task, or just messing around with the system

END OF TUTORIAL 1

# Water particle

Explain water particle code from solution branch: 

https://github.com/aidankeighron/Falling-Sand-Tutorial/tree/solution, Ignore the convert from dirt to grass
Explain how the random stuff works: 

if (getRandomInt(0, 4)) { // body } has a 75% change to run the body code 0 == false, n>0 == true

Explain how they can invert the probability with ! if (!getRandomInt(0, 4)) { // body } has a 75% change to run the body code 0 == false, n>0 == true

Task (don't give them answer and all) Have them mess around with water physics (what random range to use), extra movement options,
floating water? random change to move to a random location, have them get creative with it (add what I just mentioned as examples)
You need to add x new options?

# Swap function

Explain the swap functions and have them implement a system where sand falls beneath water

solution: https://github.com/aidankeighron/Falling-Sand-Tutorial/tree/solution

# Add more particles (Rock and Dirt)

Rock it just and extension of the base particle class with a different color and name and nothing else (you don't even need to define update or
swap functions because it inherits it from particle) DO NOT have them just create empty functions, explain the inheritance

Dirt inherits from Sand, you just need to change the color and title, the inheritance will copy the update and swap function (explain this)

# Have water convert dirt into grass

Create grass it inherits from sand, don't give them the code for this. In the update function for water before anything
else happens check if the particle below is dirt and if it is, remove the water and replace the dirt with grass:

```js
// Make water turn dirt into grass when it touches it
if (getParticle(row+1, col)?.type == "dirt") {
    // Remove water and change dirt to grass
    setParticle(row+1, col, new Grass());
    setParticle(row, col, null);
    return;
}
```

# Expand

Add x new particles.

Some suggestions and how they work

Fire:
In the particle class add a property called duration, and one called maxDuration, every time update is called increase duration, max duration
should be initialized in the constructor using a random number between x, and y. Fire works like water but moving up instead of down. And after the
duration has ended it is removed. Show code examples of duration system?

Steam:
Moves like water but upwards, has a very small chance to disappear, if it is at the top of the screen it has a change to turn into water.

+ Many more