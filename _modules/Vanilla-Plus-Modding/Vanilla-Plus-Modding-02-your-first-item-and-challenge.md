---
title: "Your First Item and Challenge"
parent_post: Vanilla-Plus-Modding
module_number: 2
layout: module
media_subpath: /assets/tutorials/minecraft-mod
---

## Your First Item

I encourage you to design your own items instead of copying mine, but if you are not interested in creating your own artwork, all my items will be provided to you.

First, make a new folder inside your mod folder. In other words, if you mod is named `tutorialcraft`, you should be putting your new folder in `com.example.tutorialcraft`.

> NOTE: Feel free to change the name of the com and example folders.
{: .prompt-info }

Name your new folder `items`.

Within your `items` folder, create the class `ModItems.java`. This will be a **central registry class** to keep track of all the items your mod wants to add.

Inside your `ModItems` class, **create a DeferredRegister** for your Forge `Items`:

```java
public static final DeferredRegister<Item> ITEMS =
    DeferredRegister.create(ForgeRegistries.ITEMS, TutorialCraft.MOD_ID);
```
{: .nolineno }
{: file="ModItems.java" }

### What is a DeferredRegister?

A DeferredRegister is Forge's **helper to add "things" to its registry**. Right now, we are using it to add items.

Specifically, it allows us to **make new items and then add them to the registry**.

Right now, `ForgeRegistries.ITEMS` holds all the built-in items before we've added our mod. This is where we will add all of our own items to be in the game.

We make it `static` so that there’s only one copy shared by the whole mod, and `final` so it never gets replaced.

### Making the Item Variable

Each item is a `RegistryObject`, which is Forge's wrapper around anything that you register into the game.

Here is an example to add a **Ruby item** into the game:

```java
public static final RegistryObject<Item> RUBY = ITEMS.register(
    "ruby",                                 // The name of the item
    () -> new Item(new Item.Properties())); // Passing the Supplier
```
{: .nolineno }
{: file="ModItems.java" }

**What does the second line do?** 

Forge wants us to **pass a Supplier to create an item** rather than just the item itself. Additionally, items have many different properties. Calling `Item.Properties()` calls all of the default properties of a typical item.

You are able to override the default properties by chaining functions. We will discuss that more later. 

### Registering Using the ModBus

In `TutorialCraft.java`, there is `modEventBus`.

**What is modEventBus?**
- An event bus that is dedicated for startup events of a mod
- This makes it useful to register things into the bus 

In the last section, we added an item into the register to define them. This portion is covering how to connect your items to the game's startup process so they actually appear in Minecraft.

In `ModItems.java`, add a function to register our `ITEMS` in a startup event:

```java
// Add this somewhere within the ModItems class
public static void register(IEventBus eventBus) {
    ITEMS.register(eventBus);
}
```
{: .nolineno }
{: file="ModItems.java" }

Next, go back to `TutorialCraft.java`. Register the ModBus with the new items inside of the **constructor**:

```java
public TutorialCraft(FMLJavaModLoadingContext context)
{
    IEventBus modEventBus = context.getModEventBus();

    ModItems.register(modEventBus); // <- Add this line
```
{: .nolineno }
{: file="TutorialCraft.java" }

### Adding the Texture

To add the item texture, we will first need to put the actual PNG of it in the correct spot. 

You should have a `resources` folder in main / src.

Add subdirectories to the resources folder so that your file tree looks like this:

```
src/
└─ main/
   └─ resources/
      └─ assets/
         └─ tutorialcraft/
            ├─ lang/
            │  └─ en_us.json
            ├─ models/
            │  └─ item/
            │     └─ ruby.json
            └─ textures/
               └─ item/
                  └─ ruby.png
```

> If you haven't turned off flattening for your file tree, I recommend you do it here. Instructions on how to do so is covered at the beginning of the tutorial.
{: .prompt-tip }

Navigate to `ruby.json` and add:

```json
{
  "parent": "item/generated",
  "textures": {
    "layer0": "tutorialcraft:item/ruby"
  }
}
```

What this means:
- `"parent" : "item/generated"` tells the game to use the built in **item/generated** model for this item, which is Mojang’s default 2D flat item model (used for things like minerals, food, enchanted books, etc.). It tells Minecraft to render this as a flat texture that always faces the player, with optional multiple texture layers.
- `"layer0": "tutorialcraft:item/ruby"` tells the game that the base texture layer is the **ruby item in our mod**

Next, we will **add to the `en_us.json` file**. This is what Minecraft uses to translate the items' keys within the mod to readable text. Specifically, this file is for English (USA). Feel free to add more languages, such as French using `fr_fr.json`, etc.

```json
{
  "item.tutorialcraft.ruby": "Ruby"
}
```

Now when look at this item in-game, its name will display as whatever you entered here.

Lastly, [download ruby.png](https://github.com/johnnystouffer/mod-tutorial/blob/main/src/main/resources/assets/tutorialcraft/textures/item/ruby.png) and move it to `resources/assets/tutorialcraft/textures/item/`.

### Add Your Item to the Menu

Last but not least, we need to add our item to the **creative menu**.

We have a function called `addCreative` in `TutorialCraft.java` that does exactly that:

```java
private void addCreative(BuildCreativeModeTabContentsEvent event)
{
    // just adding this to the Ingredients tab in the creative menu
    if (event.getTabKey() == CreativeModeTabs.INGREDIENTS) {
        event.accept(ModItems.RUBY); 
    }
}
```
{: .nolineno }
{: file="TutorialCraft.java" }


### Congrats! 

You have now created your own item! 

There are a lot of moving components, but thankfully, you won't have to repeat the setup processes for your additional items here on out.


## Challenge Item

Time to try this on your own!

Your goal is to **make a Sapphire item. However, there are additional features for you to add**:
- It needs to be fire resistant
- Stacks of Sapphires can only go up to 16
- Make its durability 500

[Here is the PNG](https://github.com/johnnystouffer/mod-tutorial/blob/main/src/main/resources/assets/tutorialcraft/textures/item/sapphire.png) for it.

**Try this on your own, using [documentation](https://docs.minecraftforge.net/en/latest/items/) for help**. View the hint below if you need it. Additionally, my solution is included if you get completely stuck.

**HINT**

```
Remember the chaining method I mentioned earlier when making the Ruby?
This applies here. Look at the documentation and try to implement it.
```
{: .nolineno }
{: .blur }

**SOLUTION**
```java
// Use chaining to edit the properties of each item
public static final RegistryObject<Item> SAPPHIRE = ITEMS.register("sapphire",
        () -> new Item(new Item.Properties()
                .fireResistant()
                .stacksTo(16)
                .durability(500))
);
```
{: .nolineno }
{: file="ModItems.java" }
{: .blur }

### Start Up the Game!

Choose the `runClient` configuration and run the game. Check inside of the **Ingredients** tab -- you should see both a Sapphire and a Ruby there.

