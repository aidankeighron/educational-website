---
title: "Your First Block"
parent_post: Vanilla-Plus-Modding
module_number: 4
layout: module
media_subpath: /assets/tutorials/minecraft-mod
---

## Your First Block

It's time to add our own blocks! Specifically, I will make the ores for the Ruby and Sapphire.

First, we need a new folder called `block` in the `tutorialcraft` folder. (`tutorialcraft` should now encase both the `/items` folder and the `block` folder.)

Within the `block` folder, make a new class called `ModBlocks.java`.

**Follow the same two steps as usual:**
- Make the `DeferredRegister` of Forge's `BLOCKS` instead of `ITEMS`
- Create a `register` method to register the `modEventBus`

I will let you do that part yourself. **The full file is included below**.

We need two more methods: `registerBlock` and `registerBlockItem`.

```java
public class ModBlocks {

    public static final DeferredRegister<Block> BLOCKS =
            DeferredRegister.create(ForgeRegistries.BLOCKS, TutorialCraft.MOD_ID);

    // Register a block: takes care of registering it with the forge blocks, as well as calling to register
    // the block as a item as well
    private static <T extends Block> RegistryObject<T> registerBlock(String name, Supplier<T> supplier) {
        RegistryObject<T> block = BLOCKS.register(name, supplier);
        registerBlockItem(name, block);
        return block;
    }

    // method to make the blocks as items as well
    private static <T extends Block> void registerBlockItem(String name, RegistryObject<T> block) {
        ModItems.ITEMS.register(name, () -> new BlockItem(block.get(), new Item.Properties()));
    }

    public static void register(IEventBus bus) {
        BLOCKS.register(bus);
    }
}
```
{: .nolineno }
{: file="ModBlocks.java" }

Make sure to add this line in the `TutorialCraft.java` class as well since we have our `register` method:

```java
public TutorialCraft(FMLJavaModLoadingContext context)
{
    IEventBus modEventBus = context.getModEventBus();

    ModCreativeModeTabs.register(modEventBus);
    ModItems.register(modEventBus);
    ModBlocks.register(modEventBus);

```
{: .nolineno }
{: file="TutorialCraft.java" }


**Why are we registering the blocks as items?**

> Blocks only exist in the world itself; the blocks you see in your inventory and creative menu are actually BlockItems. Hence, we need to register them not only as blocks for the world environment, but also as items the user can carry around and select.

### Making the Ruby Ore

The process of making the blocks is incredibly similar to making the items:
- Make a registry object of the block
- Provide the **name identifier** and **supplier** function
- Edit its properties

```java
public static final RegistryObject<Block> RUBY_ORE = registerBlock(
    "ruby_ore",
    () -> new Block(BlockBehaviour.Properties.copy(Blocks.DIAMOND_ORE)) // we want this to mock diamond ores properties
);
```
{: .nolineno }
{: file="ModBlocks.java" }


Simple as that! We also to update `en_us.json` to translate `ruby_ore`.

Additionally, you will need to **make new folders in the `resources` folder for the ruby ore block JSON file, item JSON file, and texture**. Here is what your file tree for resources should look like now:

```
src/
└─ main/
   └─ resources/
      └─ assets/
         └─ tutorialcraft/
            ├─ lang/
            │  └─ en_us.json
            ├─ models/
            │  ├─ item/
            │  │  ├─ ruby.json
            │  │  └─ ruby_ore.json
            │  └─ block/
            │     └─ ruby_ore.json
            └─ textures/
               ├─ item/
               │  └─ ruby.png
               └─ block/
                  └─ ruby_ore.png
```

Interestingly, we also need to add blockstates.

**What is a blockstate?**

> Blocks can have different appearances depending on its state, such as furnace being lit or not. Thankfully, ores only have one state.

To add blockstates, we need a new folder in our `assets/tutorialcraft` folder titled `blockstates/`.

Within that folder, create a new file called `ruby_ore.json` and add this:

```json
{
  "variants": {
    "": { "model": "tutorialcraft:block/ruby_ore" }
  }
}
```
Again, this clarifies that we only have **one variant**, so no matter the state, it displays as the one texture we provided.

Here is what `block/ruby_ore.json` should look like:

```json
{
  "parent" : "minecraft:block/cube_all",
  "textures" : {
    "all" : "tutorialcraft:block/ruby_ore"
  }
}
```
{: .nolineno }
{: file="block/ruby_ore.json" }


Ores also have the same texture around the entire block, so we can take one image and apply it to every side. Hence, the parent is `"minecraft:block/cube_all"`.

`item/ruby_ore.json` should look like:

```json
{
  "parent" : "tutorialcraft:block/ruby_ore"
}
```
{: .nolineno }
{: file="item/ruby_ore.json" }

Since we already have the block declared, we can reference it when declaring the item version.

Next, in `en_us.json` and any other languaged you are supporting, make sure to add:

```json
"block.tutorialcraft.ruby_ore" : "Ruby Ore"
```
{: .nolineno }
{: file="en_us.json" }


Next, [download the PNG](https://github.com/johnnystouffer/mod-tutorial/blob/main/src/main/resources/assets/tutorialcraft/textures/block/ruby_ore.png) for the ruby ore and put it in the `textures/block` folder.

Lastly, we can add the ruby ore item to our CreativeTab by calling this inside of the `displayItems` method:

```java
pOutput.accept(ModBlocks.RUBY_ORE.get());
```
{: .nolineno }
{: file="ModCreativeModeTabs.java" }

### Run the Client

Run the program using `runClient` to verify that the block exists. If it looks all good, then congrats -- you now have you first block!

Just like with items, adding blocks usually follows this pattern, with some variation in what you are creating .

### Challenge Block

Now lets make **Sapphire Ore**!

Complete the same steps above to create the sapphire ore. **However, there a couple things I want you to change to make this ore more interesting:**
- Make it so this ore has the friction of ice. (You slide when you walk over it)
- Make the sound of the ore sound like Amethyst
- Edit the strength of this ore
    - Set destroy time to 6.0
    - Set explostion resistance to 6.5

[Here is the](https://github.com/johnnystouffer/mod-tutorial/blob/main/src/main/resources/assets/tutorialcraft/textures/block/sapphire_ore.png) PNG for it.

> There will NOT be a solution provided for this one. There is a hint below, but you should able to comfortably figure out how to do this given this [documentation](https://docs.minecraftforge.net/en/latest/blocks/#creating-a-block) and Google (try your best not to use AI right now).
{: .prompt-danger }

```
Remember the chaining you did for the item properties? 
Try that again! Typing a period should show you a list of functions you can use as well.
```
{: .blur }
{: .nolineno }


