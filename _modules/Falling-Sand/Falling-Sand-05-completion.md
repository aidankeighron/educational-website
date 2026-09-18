---
title: "Completion"
parent_post: Falling-Sand
module_number: 5
layout: module
media_subpath: /assets/tutorials/falling sand
---

# Completion & Discussion Checklist

Before joining the group discussion or concluding this tutorial, ensure you have completed the tasks, investigated the bugs, and are ready to discuss the questions below:

<details markdown="1">
<summary>Click to expand Completion & Discussion Checklist (8 Items)</summary>

| # | Type | Item | Prompt Preview |
| :-: | :--- | :--- | :--- |
| 1 | Bug Hunt | Vanishing Water Swap Bug | Run the simulation and drop sand on water. The sand falls through, but the water vanishes! Why did we lose the water particle, and how can we use `temp` to exchange their positions? |
| 2 | Question | Random Probabilities | Try making water have a small chance to move upwards. What parameters for `getRandomInt()` would you use for a very low probability? |
| 3 | Question | Elemental Simulation Rules | Think about other substances and elements. How can you define interaction rules for Acid, Ice, Lava, or Steam within object-oriented subclasses? |
| 4 | Task | Water Movement Variations | Mess around with water physics! Change movement probabilities, add extra options, make floating water, or add teleportation. Add 3 new behaviors to water's `update` function. |
| 5 | Task | Create `Stone` Class | Create a new class called `Stone` that extends `Particle`. In its constructor, set color to `"gray"` and type to `"stone"`. Add `Stone` as an option in `checkParticleType`. |
| 6 | Task | Create `Dirt` Class | Create a new class called `Dirt` that extends `Sand`. In its constructor, set color to `"brown"` and type to `"dirt"`. Add `Dirt` as an option in `checkParticleType`. |
| 7 | Task | Create `Grass` Class | Create a new class called `Grass` extending `Sand` (`color: "green"`, `type: "grass"`). Do not add it to `checkParticleType` — grass can only be created when water touches dirt. |
| 8 | Challenge | 3 Custom Particles | Add at least 3 new particles and make sure to add interactions with other particles (don't just add `Metal` and make it act like `Stone`). Get creative with it! |

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
