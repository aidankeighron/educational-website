---
title: "Tools and Armor"
parent_post: Vanilla-Plus-Modding
module_number: 5
layout: module
media_subpath: /assets/tutorials/minecraft-mod
---

## Making Your Own Tools

Lets try making our own tools, too!

We have made items before, and as mentioned before, **items are for your inventory and creative mode tabs**, so tools / weapons are also items. However, they have more functionality than other items that are just in your inventory, so they have their **own subclasses**. It is rather simple in practice. For example, instead of using `Item.Properties()`, you would call `SwordItem.Properties()`. 

There are also different ways to customize these items, which will be covered.

We will go over creating a custom sword item, after which you will be able to create the rest of the tools on your own.

### Create a Sword

**We need to add the following for every new item:**
- The PNG for the item texture
- Translation from its item ID to its language value in `en_us.json`
- A JSON file for the item's appearance
- Add the item and register it
- Add it to the creative mode menu

A quick overview of each step is provided, going in more detail on the new / different things, but you should be getting pretty comfortable with creating your own items now.

#### Add the PNG for the Item Texture

[Here is the link](https://github.com/johnnystouffer/mod-tutorial/blob/main/src/main/resources/assets/tutorialcraft/textures/item/ruby_sword.png) to the ruby sword's PNG. Download it, then add it to `resources/assets/tutorialcraft/textures/item`.

#### Set the Item ID's Language Value

Create a new variable in the `resources/assets/tutorialcraft/lang/en_us.json` file.

**For example:**
```json
"item.tutorialcraft.ruby_sword" : "Ruby Sword"
```

#### Make the JSON File for the Item's Appearance

This will look nearly **identical** to the ones you made for Ruby and Sapphire. There is only one key difference: *tools and weapons are handheld items, not flat generated ones*.

If you do not understand what the above means, I suggest you search Google for the answer. Try your best to find resources -- there are many out there.

If you are struggling to find any answers, [read this](https://forums.minecraftforge.net/topic/40899-sword-json-file-help-1102/).

#### Add the Item and Register It

Since we already have our `ModItems.java` class, we register the ruby sword there to avoid creating new, unnecessary register functions.

> TRY TO DO THIS YOURSELF. I have given enough hints and I am confident you can figure out this code on your own, especially if you use Google and the documentation. I will show what I have below still (WITH ERRORS), but please try it on your own. 
{: .prompt-warning }

```java
// Add this into your ModItems.java
public static final RegistryObject RUBY_SWORD   = ITEMS.register("ruby_sword",
    () -> new Sword(
        DiamondTier,            // Tier of the weapon (Netherite being the best and Wood being the worst)
        3,                      // Attack Damage Modifier. (final damage = tier damage + this number)
        -2.4,                  // Attack speed modifier (lower = slower, -2.4F is standard for a sword)
        new Item.properties
    )
);

// Add this in ModCreativeModeTabs.java to add it to the inventory
pOutput.accept(ModItems.RUBY_SWORD.get());
```
{: .nolineno }
{: .blur }

> **TIP**: If you go to `.minecraft/versions/[version]/[version].jar` on your own system, you can find all of Minecraft's built-in assets. You will be able to view the JSON files for how certain items are made.
{: .prompt-tip }

### Create the Rest of Your Tool Set

Now that you have created items, a sword, and have access to resources within your own systems and on google, **do your best to make the rest of the tool set**. Here is the art for your convenience:
- [Ruby Pickaxe](https://github.com/johnnystouffer/mod-tutorial/blob/main/src/main/resources/assets/tutorialcraft/textures/item/ruby_pickaxe.png)
- [Ruby Axe](https://github.com/johnnystouffer/mod-tutorial/blob/main/src/main/resources/assets/tutorialcraft/textures/item/ruby_axe.png)
- [Ruby Shovel](https://github.com/johnnystouffer/mod-tutorial/blob/main/src/main/resources/assets/tutorialcraft/textures/item/ruby_shovel.png)
- [Ruby Hoe](https://github.com/johnnystouffer/mod-tutorial/blob/main/src/main/resources/assets/tutorialcraft/textures/item/ruby_hoe.png)


## Making Your Own Armor 

This section marks the last bit of instruction in this tutorial before we leave it up to you... **Lets make some armor!**

The only code covered will be what goes in `ModItems`; you should be more than fine doing the rest on your own using previous examples, documentation, Google, etc.

```java
public static final RegistryObject<Item> RUBY_HELMET = ITEMS.register(
    "ruby_leggings",
    () -> new ArmorItem(
        "ruby",                  // material used IMPORTANT FOR THE TEXTURES
        ArmorItem.Type.HELMET,   // Type of armror
        new Item.Properties()));

```
{: .nolineno }
{: .blur }
{: file="ModItems.java" }

### Wearing the Armor

> **QUESTION:** Why is specifying the exact material name for armor so critical in Minecraft Forge? What naming pattern does Forge expect for wearable armor texture files?
{: .prompt-tip }

> **NOTE:** Forge appends `_layer_1.png` and `_layer_2.png` to your provided material name to locate the wearable model textures. Your material string **MUST** match your PNG file names exactly! Put these texture files in `src/main/resources/assets/<modid>/textures/models/armor/`.
{: .prompt-info }

Here is a [link to the two PNGs](https://github.com/johnnystouffer/mod-tutorial/tree/main/src/main/resources/assets/tutorialcraft/textures/models/armor) for you to add.

### Finish the Armor Set

You should have more than enough resources by now to make the rest of the armor set.

Feel free to finish it up, make a different one with Sapphire, or design your own. The sky is the limit!

