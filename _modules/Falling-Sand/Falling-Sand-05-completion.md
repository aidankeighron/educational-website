---
title: "Completion"
parent_post: Falling-Sand
module_number: 5
layout: module
media_subpath: /assets/tutorials/falling sand
---

## Completion & Discussion Checklist

Before joining the group discussion or concluding this tutorial, ensure you have completed the tasks, investigated the bugs, and are ready to discuss the questions below:

<details markdown="1">
<summary>Click to expand Completion & Discussion Checklist (15 Items)</summary>

| # | Type | Item | Prompt Preview |
| :-: | :--- | :--- | :--- |
| 1 | Bug Hunt | Empty Grid Null Check | The code we just added has an error. `grid` uses `null` to represent empty spaces. This will cause an error when we try to access `particle.color` because `particle` is `null`. |
| 2 | Bug Hunt | Vanishing Water Swap Bug | Run the simulation, create a pool of water, and drop sand particles on top of it. Notice anything strange? The sand falls through, but the water completely vanishes into thin air instead of rising to the surface! |
| 3 | Question | Grid Bounds & Coordinates | Think about the dimensions of our grid. How can you check if a given row is within the valid range of rows? What about the column? |
| 4 | Question | Particle Reference vs. Copy | What do we use to represent an empty particle? Are we moving the particle or just making a new one? |
| 5 | Question | Random Probabilities | Floating water? Try making water have a small chance to move upwards. What parameters for `getRandomInt()` would you use for a very low probability? |
| 6 | Question | Elemental Simulation Rules | Think about other substances and elements. How can you define interaction rules for Acid, Ice, Lava, or Steam within object-oriented subclasses? |
| 7 | Task | Implement `checkBounds()` | Modify the `checkBounds` function so it returns `true` if `(row, col)` is within the bounds of `grid` and `false` otherwise. After writing the function, use it in `moveParticle` to prevent particles from being moved out of bounds. |
| 8 | Task | Fix Particle Streaking | Modify the `moveParticle` function in `canvas.js` to stop the particles from streaking as they fall. |
| 9 | Task | Prevent Particle Overwrite | Utilizing `getParticle` (which returns the particle at `(row, col)`), add a check in the `moveParticle` function in `canvas.js` to make sure a particle cannot move on top of another particle. |
| 10 | Task | Water Movement Variations | Mess around with water physics! Change the probabilities of movement, add extra movement options, make floating water, or add a random chance to teleport to a random location. Add **`3`** new behaviors to water's `update` function. |
| 11 | Task | Create `Stone` Class | Create a new class called `Stone` that extends the `Particle` class. In its constructor, set the color to `"gray"` and the type to `"stone"`. Make sure to add `Stone` as an option in `checkParticleType`. |
| 12 | Task | Create `Dirt` Class | Create a new class called `Dirt` that extends the `Sand` class. In its constructor, set the color to `"brown"` and the type to `"dirt"`. Make sure to add `Dirt` as an option in `checkParticleType`. |
| 13 | Task | Create `Grass` Class | Create a new class called `Grass` in `particles.js` that extends the `Sand` class. In its constructor, set the color to `"green"` and the type to `"grass"`. Grass can ONLY be created when water touches dirt. |
| 14 | Challenge | Custom Physics Experimentation | Mess around with the sand physics! What happens if you have the sand move two steps every update (`row+2` or `col+2`), or if you try to move left and right first? |
| 15 | Challenge | 3 Custom Particles | Add at least `3` new particles and make sure to add interactions with other particles (don't just add `Metal` and make it act like `Stone`). Get creative with it! |

</details>


# Congratulations!

You've now taken your Falling Sand simulation to the next level! In this second part of the tutorial, you've successfully:

- Introduced a new particle type, Water, and implemented its unique physics, including random movement and interactions with other particles.
- Utilized the getRandomInt() function to add probabilistic behavior to your particles, making the simulation more dynamic and realistic.
- Implemented the swap() function to create interactions between different particle types, allowing Sand to fall through Water.
- Added Stone and Dirt particles, demonstrating how to extend existing classes and create new particle behaviors.
- Created a dynamic interaction between Water and Dirt, resulting in the creation of Grass, showcasing the power of particle interactions.
- Further expanded your understanding of JavaScript classes, inheritance, and object-oriented programming.
- Practiced your problem-solving and debugging skills by experimenting with particle behaviors.

**You've built a solid foundation for creating even more complex and interesting particle simulations. Now, let's explore how you can further expand your project with new particles and interactions!**
