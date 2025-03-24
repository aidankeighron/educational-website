---
title: Falling Sand 2
author: aidan
date: 2025-03-23 12:00:00 +0800
categories: [JavaScript, FallingSand]
tags: [JavaScript, Easy]
description: Part 2 of the Falling Sand project. Creating new particles, and adding new interactions.
comments: false
pin: true
---

## Water particle

Now let's add a new particle type: Water!

For water we will be using random numbers a lot. We have a `getRandomInt(min, max)` function in `util.js` that will be very helpful. It generates a random number between `min` and `max` inclusive (min and max are possible numbers). This is used to give a snippet of code a random chance to run or not run.

> Remember `0` is `false` and any other number, positive or negative is `true`.
{: .prompt-info }

```js
if (getRandomInt(0, 4)) { 
    // 75% Chance to run
}

if (!getRandomInt(0, 3)) { 
    // 33% Chance to run
}
```

Using this lets write some basic physics for a water particle. Add this class below `Sand` in `particles.js`:

```js
export class Water extends Particle {
    constructor() {
        super();
        this.color = "blue";
        this.type = "water";
    }

    update(row, col) {
        // Try to move down
        if (getRandomInt(0, 2) && !getParticle(row+1, col)) {
            moveParticle(row, col, row+1, col, super.swap);
        } 
        
        // Move left or right
        if (getRandomInt(0, 1) && !getParticle(row, col+1)) {
            moveParticle(row, col, row, col+1, super.swap);
        }
        else if (!getParticle(row, col-1)) {
            moveParticle(row, col, row, col-1, super.swap);
        }
    }
}
```
{: file="particles.js" }
{: .nolineno }

And add water as an option when creating a particle

```js
/**
 * Create particle based on dropdown name
 * 
 * @param {string} value 
 * @returns 
 */
export function checkParticleType(value) {
    if (value == "Sand") {
        return new Sand();
    }
    // 👇 Put the following code here 👇
    if (value == "Water") {
        return new Water();
    }
    // 👆 Put the following code here 👆
}
```
{: file="particles.js" }
{: .nolineno }

Now when you switch the particle dropdown to `Water` you will be able to create water particles.

> Task: Mess around with water physics change the probabilities of movement, add extra movement options, make floating water? Random chance to move to a random location. Add **`3`** new things to waters `update` function
{: .prompt-tip }


<details>
<summary>Suggestion 1</summary>
<blockquote>
Add more movement options: Currently, water tries to move down, then left or right. Try adding a small chance for it to move diagonally down-left or down-right.
</blockquote>
</details>


<details>
<summary>Suggestion 2</summary>
<blockquote>
Floating water? Try making water have a small chance to move upwards. What parameters for getRandomInt() would you use for a very low probability?
</blockquote>
</details>


<details>
<summary>Suggestion 3</summary>
<blockquote>
Random change to move to a random location: This is a more advanced challenge! Try giving water a very small chance to teleport to a completely random location on the grid.
</blockquote>
</details>

<br>

Have fun experimenting and see what kind of interesting water behaviors you can create!

## Swap function

Now let's implement the `swap()` function in our `particles.js` file. This will allow us to define how different particles interact when they try to move into the same space.

Currently, if sand tries to move into a space occupied by water, it will just stop and float on top of the water. We want to make it so that sand falls beneath water. To do this, we need to implement the `swap()` method in the Sand class. This will return `true` when a particle is allowed to swap with another when trying to move on to a grid location occupied by `other`.

Change the `swap()` function in the `Sand` class to look like this:


```js
swap(other) {
    // Make sand fall below water
    return other.type == "water";
}
```
{: file="particles.js" }
{: .nolineno }

Now, lets modify the `moveParticle()` function to use the `swap` argument.

```js
export function moveParticle(row, col, newRow, newCol, swap) {
    if (!checkBounds(row, col) || !checkBounds(newRow, newCol)) {
        return false;
    }

    if (getParticle(newRow, newCol)) {
        // 👇 Add this check and swap logic 👇
        // If there is a particle but we can swap then flip the particles
        if (swap && swap(getParticle(newRow, newCol))) {
            const temp = grid[newRow][newCol];
            grid[newRow][newCol] = grid[row][col];
            grid[row][col] = temp;
            return true;
        }
        // If we can't swap then don't move
        else {
            return false;
        }
        // 👆 Add this check and swap logic 👆
    }

    grid[newRow][newCol] = grid[row][col];
    grid[row][col] = null;
    return true;
}
```


The `swap &&` in `if (swap && swap(getParticle(newRow, newCol))) {` just makes sure swap is not `undefined` or `null`.

## Add more particles (Stone and Dirt)

Let's add two more particle types to our simulation: `Stone` and `Dirt`.

Stone is just an extension of the base particle class with a different color and name (you don't even need to define update or swap functions because it inherits it from Particle).

> Task: Create a new class called `Stone` that extends the Particle class. In its constructor, set the color to `"gray"` and the type to `"stone"`. You don't need to add any `update()` or `swap()` methods to the `Stone` class. Make sure to add `Stone` as an option in `checkParticleType`.
{: .prompt-tip }

**Answer (click to unblur):**

```js
/**
 * Rock particle
 */
export class Stone extends Particle {
    constructor() {
        super(); // Call the constructor of the Particle class
        this.color = "gray";
        this.type = "stone";
    }
    // No update or swap needed!
}
```
{: file="particles.js" }
{: .nolineno }
{: .blur }

`Dirt` inherits from `Sand`, you just need to change the color and title, the inheritance will copy the `update` and `swap` function.

> Task: Create a new class called `Dirt` that extends the Sand class. In its constructor, set the color to `"brown"` and the type to `"dirt"`. You don't need to add any `update()` or `swap()` methods to the `Dirt` class. Make sure to add `Dirt` as an option in `checkParticleType`.
{: .prompt-tip }

**Answer (click to unblur):**

```js
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
```
{: file="particles.js" }
{: .nolineno }
{: .blur }

If you have not done so yet update the `checkParticleType()` function to create and return instances of the `Stone` and `Dirt` classes when their corresponding values are selected in the dropdown.

```js
export function checkParticleType(value) {
    if (value == "Sand") {
        return new Sand();
    } else if (value == "Water") {
        return new Water();
    } else if (value == "Stone") { // Add this
        return new Stone();
    } else if (value == "Dirt") {   // Add this
        return new Dirt();
    }
    return null;
}
```
{: file="particles.js" }
{: .nolineno }

Now you should be able to select `Stone` and `Dirt` from the dropdown and create them in your simulation! `Stone` will stay in place, and `Dirt` will fall like sand.

## Have water convert dirt into grass

Let's add an interesting interaction: when water touches dirt, it will turn the dirt into grass!

> Task: Create a new class called `Grass` in `particles.js` that extends the `Sand` class. In its constructor, set the color to `"green"` and the type to `"grass"`. **DO NOT** add the particle as an option in `checkParticleType()`. Grass can ONLY be created with water and dirt.
{: .prompt-tip }


Now, let's implement the logic for water turning dirt into grass. Go to the waters `update` function and add this code:

```js
update(row, col) {
    // 👇 Put the following code here 👇
    // Make water turn dirt into grass when it touches it
    if (getParticle(row+1, col)?.type == "dirt") {
        // Remove water and change dirt to grass
        setParticle(row+1, col, new Grass());
        setParticle(row, col, null);
        return;
    }
    // 👆 Put the following code here 👆

    // Try to move down
    if (getRandomInt(0, 2) && !getParticle(row+1, col)) {
        moveParticle(row, col, row+1, col, this.swap);
    } 
    
    // Move left or right
    if (getRandomInt(0, 1) && !getParticle(row, col+1)) {
        moveParticle(row, col, row, col+1, this.swap);
    }
    else if (!getParticle(row, col-1)) {
        moveParticle(row, col, row, col-1, this.swap);
    }
}
```
{: file="particles.js" }
{: .nolineno }

Now, try creating some dirt and then some water on top of it – you should see the dirt turn into grass!

## Next Steps


### Adding new particles

There are `3` things you need to do to add a new particle

- Create a new class that inherits from `Particle` or is a subclass of `Particle`
- Add the particle as an option in `checkParticleType`
- Add the particle to the `type` dropdown in `index.html`

Example of adding a new particle in `index.html`. `value` is the string that you use in `checkParticleType`, the text between the `option` tags is the text that is displayed on the dropdown.

```html
<!-- the rest of index.html -->

<option value="Sand">Sand</option>
<option value="Water">Water</option>
<option value="Stone">Stone</option>
<option value="Dirt">Dirt</option>
<!-- <option value="Fire">Fire</option> -->
<!-- <option value="Wood">Wood</option> -->
<!-- <option value="Steam">Steam</option> -->
```
{: file="index.html" }
{: .nolineno }


Here are a couple of ideas for new particles you can add to your simulation:

#### Fire

For the `Fire` particle you need to keep track of duration and max duration, every time update is called increase duration. Max duration should be initialized in the constructor  and once duration is `>=` to max duration is is remove (or has a chance to be removed). Fire works like water but moving up instead of down.

#### Wood

Acts like `Stone` but gets destroyed by fire and creates more of it. Maybe it absorbs water and that makes it harder to burn?

#### Steam

Moves like water but upwards, has a very small chance to disappear, if it is at the top of the screen it has a chance to turn condense into water.

Think about other real-world substances or fantastical elements and how they might behave. Could you add:

- Acid: A particle that falls and dissolves other particles it touches.
- Ice: A particle that falls and can freeze water.
- Lava: A particle that falls and can burn other particles.


> Task: Add at least `3` new particles and make sure to add interactions with other particles (don't just add `Metal` and make it act like `Stone`). Get creative with it.
{: .prompt-tip }