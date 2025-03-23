---
title: Falling Sand 2
author: aidan
date: 2025-03-21 12:00:00 +0800
categories: [JavaScript, FallingSand]
tags: [JavaScript, Easy]
description: Part 2 of the Falling Sand project.
comments: false
pin: true
---

## Water particle

Now let's add a new particle type: Water!

For water we will be using random numbers a lot. We have a `getRandomInt(min, max)` function in `util.js` that will be very helpful. It generates a random number between `min` and `max` inclusive (min and max are possible numbers). This is used to give a smippit of code

```js
if (getRandomInt(0, 4)) { 
    // 75% Chance to run
}

if (!getRandomInt(0, 3)) { 
    // 33% Chance to run
}
```

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