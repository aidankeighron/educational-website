---
title: "Making Your Own Creative Tab"
parent_post: Vanilla-Plus-Modding
module_number: 3
layout: module
media_subpath: /assets/tutorials/minecraft-mod
---

## Making Your Own Creative Tab

We do not just want to dump everything into Ingredients, since that will lump all your custom items with existing ones. So, lets make tab dedicated to your mod's items.

### Setting Up the Tab

Add a new class inside of the `/items` folder and call it `ModCreativeModeTabs.java`.

Like the items above, **we need to add a DeferredRegister into this class, this time of the Creative Mode Tabs**. Additionally, we need to **make a register function call for the modEventBus**.

```java
public static final DeferredRegister<CreativeModeTab> CREATITVE_MODE_TABS =
    DeferredRegister.create(Registries.CREATIVE_MODE_TAB, TutorialCraft.MOD_ID);

public static void register(IEventBus eventBus) {
    CREATITVE_MODE_TABS.register(eventBus);
}
```
{: .nolineno }
{: file="ModCreativeModTabs.java" }

Head back to the main mod class `TutorialCraft.java`. **Register the Creative Mod Tab with the mod event bus** around the same place you did for the `ModItems`.

```java
ModCreativeModeTabs.register(modEventBus);
```
{: .nolineno }

### Making the Tab

Just like before, we are making a `RegistryObject`, but this time it is a `CreativeMobTab` registry object.

For our `Supplier` parameter, we will use the builder pattern by using the `CreativeModTab.builder()` function.

**With this, we can customize the tab to our liking. In this tutorial, we will customize the following:**
- **Icon**: The icon that shows up for the tab
- **Title**: What the tab is called
- **Display items**: The items you want in the tab

To do this, reference the the function below:

```java
public static final RegistryObject<CreativeModeTab> TUTORIAL_TAB = CREATITVE_MODE_TABS.register("tutorial_tab",
    () -> CreativeModeTab.builder()
        // add a icon to the tab with that being the Ruby item we made
        .icon(() -> new ItemStack(ModItems.RUBY.get()))
        // add a title to the tab
        .title(Component.translatable("creativetab.tutorial_tab"))
        // display the Ruby and Sapphire we just made
        .displayItems((pParameters, pOutput) -> {
            pOutput.accept(ModItems.RUBY.get());
            pOutput.accept(ModItems.SAPPHIRE.get());
        })
        .build());
```
{: .nolineno }
{: file="ModCreativeModTabs.java" }


> Notice the title has `Component.translatable("creativetab.tutorial_tab")`. This makes the title **dynamic** based on the language, just like with the Ruby and Sapphire. In `en_us.json`, add the variable **("creativetab.tutorial_tab")** and set the value as whatever you want to call your tab.

Now, any time we add a new item to the mod, we can keep it in our own tab.

