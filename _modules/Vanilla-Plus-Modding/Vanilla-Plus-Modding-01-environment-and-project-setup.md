---
title: "Environment and Project Setup"
parent_post: Vanilla-Plus-Modding
module_number: 1
layout: module
media_subpath: /assets/tutorials/minecraft-mod
---

## Setting Up Your Environment

### JDK 17

The first thing you need installed is the proper Java Development Kit (JDK). Specifically, you will need **JDK 17**. 

*Why JDK 17 specifically?* 

> We are using Minecraft version 1.20.1, and this version of minecraft uses JDK 17. Hence, other JDK versions will not work.

*Why are we using 1.20.1 when newer versions are out?*

> The goal is for the setup process to be as easy as possible for everyone. MacOS seemed to have trouble with newer versions, while this version worked right out of the box.

To download the JDK 17, [click here](https://adoptium.net/temurin/releases/?version=17). Download based on your OS and go through all the prompts it gives you. Now, you should be good to go!

 > I will not cover downloading the Java Runtime Environment (JRE) or other Java Concepts. If you are unaware of what the JRE is or do not have prior Java experience, I HIGHLY suggest you learn Java before moving forward. 
 {: .prompt-warning }

### Download the Mod Pack

Next, we need to download the actual **mod development kit (MDK)**. [Click here](https://files.minecraftforge.net/net/minecraftforge/forge/index_1.20.1.html) to visit the download page.

You will see two options: **Latest** and **Recommended**. Both should work fine, but the safer route is, of course, **Recommended**. 

Within these two options, you will also see:
- A big box labeled `Installer`
- Top right box labeled `Changelog`
- Bottom right box labeled `Mdk`

> Click the box that says `Mdk`, this will take you to a screen of *a large ad*. In other words, **DO NOT CLICK DOWNLOAD. IT IS A SCAM / VIRUS**. Instead, click **Skip Ad** on the top right corner when it shows up and the MDK will download after that.
{: .prompt-danger}

> Sadly, you will likely run into similar ads very often when modding with Minecraft, whether it be while downloading a new mod, resource pack, shader pack, etc. I highly recommend a good ad blocker to avoid them.
{: .prompt-info}


After you skip the ad, it will download **forge-1.20.1-47.4.0-mdk.zip**.

Unpack this folder and rename it to whatever you want your mod to be called.

**This is optional, but there are some things you can delete within the folder:**

- changelog.txt
- CREDITS.txt
- LICENSE.txt
- README.txt


### Setting Up Intellij

> You should still go through this section even if you are using a different IDE. It will cover changes to gradle.properties, settings.gradle, etc. However, the actual running of the project and using Gradle is completely on you.
{: .prompt-info}

Ensure you have either the *full version* or *Community Edition* of Intellij installed.

When you launch Intellij, click `Open`.

Select the **unzipped Mdk** to open. Go through the prompts. It may ask if you *Trust this Project?* in which you should select "yes". 

Now that it is open, you want to make sure you have the right JDK chosen. 

- Go to **Settings > Build, Execution, Deployment > Build Tools > Gradle**
- You will see **Gradle JVM** with a JDK already selected. Make sure it is `temurin-17`, as that is what we downloaded.

> If you do not see anything selected / there is no option for `temurin-17`, something likely went wrong with your installation of the Java Development Kit. Try the steps again to install the JDK, ensuring the installation is in your PATH and JAVA_HOME variables, etc.
{: .prompt-warning}

Let's go back to the main window of your project. You will see the **Gradle logo (an elephant)**. Click on it to open a new window/tab, then use the refresh icon at the top left corner of the tab to build the project.

There should be **no errors** at this time. You should see a **couple warnings**, but those are not an issue as of now.

> **Tip:** Right-click on the file tree, click *Appearence*, then disable *Flatten Modules* and *Flatten Packages*. This will make it so it shows ALL folders instead of combining folders like so: `com.folder1.folder2`
{: .prompt-tip}


## Setting up the Mod Project

**Congrats!** The hardest part of this tutorial should be over. I'd be both impressed and jealous if you managed to get here with no errors or setbacks.

First, navigate to `ExampleMod.java`. You will find a whole lot of jargon that we do not need.

**Feel free to remove:**
- Any variable under `MODID` and `LOGGER`
- The function bodies of `commonSetup`, `addCreative`, `onServerStarting`, and `onClientSetup`

The contents of the file should look like this:

```java
@Mod(ExampleMod.MOD_ID)
public class ExampleMod
{
    // Define mod id in a common place for everything to reference
    public static final String MOD_ID = "examplemod";
    // Directly reference a slf4j logger
    private static final Logger LOGGER = LogUtils.getLogger();

    public ExampleMod(FMLJavaModLoadingContext context)
    {
        IEventBus modEventBus = context.getModEventBus();

        // Register the commonSetup method for modloading
        modEventBus.addListener(this::commonSetup);

        // Register ourselves for server and other game events we are interested in
        MinecraftForge.EVENT_BUS.register(this);

        // Register the item to a creative tab
        modEventBus.addListener(this::addCreative);

        // Register our mod's ForgeConfigSpec so that Forge can create and load the config file for us
        context.registerConfig(ModConfig.Type.COMMON, Config.SPEC);
    }

    private void commonSetup(final FMLCommonSetupEvent event)
    {
    }

    // Add the example block item to the building blocks tab
    private void addCreative(BuildCreativeModeTabContentsEvent event)
    {
    }

    // You can use SubscribeEvent and let the Event Bus discover methods to call
    @SubscribeEvent
    public void onServerStarting(ServerStartingEvent event)
    {
    }

    // You can use EventBusSubscriber to automatically register all static methods in the class annotated with @SubscribeEvent
    @Mod.EventBusSubscriber(modid = MOD_ID, bus = Mod.EventBusSubscriber.Bus.MOD, value = Dist.CLIENT)
    public static class ClientModEvents
    {
        @SubscribeEvent
        public static void onClientSetup(FMLClientSetupEvent event)
        {
        }
    }
}
```
{: file="ExampleMod.java" }
{: .nolineno }

### Update Mod ID and Gradle files

#### Mod ID

The file above has the `MOD_ID` variable. This ID is for uniquely identifying your mod, but it has specific requirements: **It can only contain lowercase letters, numbers, underscores, or dashes**. I named mine to `tutorialcraft`, so keep that in mind when seeing my code.

```java
public static final String MOD_ID = "tutorialmod";
```
{: .nolineno}

#### Mod Class Name

The class name and file name is still `ExampleMod`. You can rename these to be like your `MOD_ID`, except **the class and file name can contain uppercase letters**. I changed mine to be `TutorialCraft`.

There are quite a few areas where the class name is used. The safest and quickest way to change it is:

**Highlight the current class name > right click > click refactor.** 
 
There will be an error because your file name must be the same as the name of the class inside, so make sure to update your file name accordingly.

#### Fix the Configuration File

You might have noticed that your config file has errors around these lines:

```java
new ResourceLocation(itemName)
```
{: .nolineno }

That is because the new version of Forge replaced its public constructors for `ResourceLocation` with static methods.

Change all those instances to:

```java
ResourceLocation.tryParse(itemName)
```
{: .nolineno }

#### Your Files at This Stage

> **Class import statements were not included** in the following code.
{: .prompt-warning}

```java
@Mod.EventBusSubscriber(modid = TutorialCraft.MOD_ID, bus = Mod.EventBusSubscriber.Bus.MOD)
public class Config
{
    private static final ForgeConfigSpec.Builder BUILDER = new ForgeConfigSpec.Builder();

    private static final ForgeConfigSpec.BooleanValue LOG_DIRT_BLOCK = BUILDER
            .comment("Whether to log the dirt block on common setup")
            .define("logDirtBlock", true);

    private static final ForgeConfigSpec.IntValue MAGIC_NUMBER = BUILDER
            .comment("A magic number")
            .defineInRange("magicNumber", 42, 0, Integer.MAX_VALUE);

    public static final ForgeConfigSpec.ConfigValue<String> MAGIC_NUMBER_INTRODUCTION = BUILDER
            .comment("What you want the introduction message to be for the magic number")
            .define("magicNumberIntroduction", "The magic number is... ");

    // a list of strings that are treated as resource locations for items
    private static final ForgeConfigSpec.ConfigValue<List<? extends String>> ITEM_STRINGS = BUILDER
            .comment("A list of items to log on common setup.")
            .defineListAllowEmpty("items", List.of("minecraft:iron_ingot"), Config::validateItemName);

    static final ForgeConfigSpec SPEC = BUILDER.build();

    public static boolean logDirtBlock;
    public static int magicNumber;
    public static String magicNumberIntroduction;
    public static Set<Item> items;

    private static boolean validateItemName(final Object obj)
    {
        return obj instanceof final String itemName && ForgeRegistries.ITEMS.containsKey( ResourceLocation.tryParse(itemName));
    }

    @SubscribeEvent
    static void onLoad(final ModConfigEvent event)
    {
        logDirtBlock = LOG_DIRT_BLOCK.get();
        magicNumber = MAGIC_NUMBER.get();
        magicNumberIntroduction = MAGIC_NUMBER_INTRODUCTION.get();

        // convert the list of strings into a set of items
        items = ITEM_STRINGS.get().stream()
                .map(itemName -> ForgeRegistries.ITEMS.getValue(ResourceLocation.tryParse(itemName)))
                .collect(Collectors.toSet());
    }
}
```
{: .nolineno }
{: file="Config.java" }

```java
@Mod(TutorialCraft.MOD_ID)
public class TutorialCraft
{
    // Define mod id in a common place for everything to reference
    public static final String MOD_ID = "tutorialmod";
    // Directly reference a slf4j logger
    private static final Logger LOGGER = LogUtils.getLogger();

    public TutorialCraft(FMLJavaModLoadingContext context)
    {
        IEventBus modEventBus = context.getModEventBus();

        // Register the commonSetup method for modloading
        modEventBus.addListener(this::commonSetup);

        // Register ourselves for server and other game events we are interested in
        MinecraftForge.EVENT_BUS.register(this);

        // Register the item to a creative tab
        modEventBus.addListener(this::addCreative);

        // Register our mod's ForgeConfigSpec so that Forge can create and load the config file for us
        context.registerConfig(ModConfig.Type.COMMON, Config.SPEC);
    }

    private void commonSetup(final FMLCommonSetupEvent event)
    {
    }

    // Add the example block item to the building blocks tab
    private void addCreative(BuildCreativeModeTabContentsEvent event)
    {
    }

    // You can use SubscribeEvent and let the Event Bus discover methods to call
    @SubscribeEvent
    public void onServerStarting(ServerStartingEvent event)
    {
    }

    // You can use EventBusSubscriber to automatically register all static methods in the class annotated with @SubscribeEvent
    @Mod.EventBusSubscriber(modid = MOD_ID, bus = Mod.EventBusSubscriber.Bus.MOD, value = Dist.CLIENT)
    public static class ClientModEvents
    {
        @SubscribeEvent
        public static void onClientSetup(FMLClientSetupEvent event)
        {
        }
    }
}
```
{: file="TutorialCraft.java" }
{: .nolineno }

### Parchment Mappings

Mojang obfuscates code before they release a new version of Minecraft, so we will use **ParchmentMC mappings** to make it readable.

> Although Mojang does release their own mappings, they are usually incomplete, which is why Parchment mappings became the standard.
{: .prompt-info }

There are a couple steps needed for you to use ParchmentMC mappings.

First, add this segment into `settings.gradle`:

```groovy
 pluginManagement {
     repositories {
         maven { url = 'https://maven.parchmentmc.org' } // Add this line
     }
 }
```
{: .nolineno }


Then, update these segments in your `build.gradle`:

```groovy
 plugins {
     // This should be below the net.minecraftforge.gradle plugin
     id 'org.parchmentmc.librarian.forgegradle' version '1.+'
 }

  minecraft {
    // update this line (first line under minecraft)
     mappings channel: 'parchment', version: '2023.09.03-1.20.1'
 }
```
{: .nolineno }


Lastly, update these two lines in `gradle.properties`:

```properties
mapping_channel=parchment
mapping_version=2023.09.03-1.20.1
```
{: .nolineno }

Feel free to change any of the other lines in `gradle.properties` as well. This is where you determine the author, version, description, etc. 

For example:

```properties
# The license of the mod. Review your options at https://choosealicense.com/. All Rights Reserved is the default.
mod_license=All Rights Reserved
# The mod version. See https://semver.org/
mod_version=1.0.0
# The group ID for the mod. It is only important when publishing as an artifact to a Maven repository.
# This should match the base package used for the mod sources.
# See https://maven.apache.org/guides/mini/guide-naming-conventions.html
mod_group_id=com.example.tutorialcraft
# The authors of the mod. This is a simple text string that is used for display purposes in the mod list.
mod_authors=JohnStouffer
# The description of the mod. This is a simple multiline text string that is used for display purposes in the mod list.
mod_description=Make your own MC Mod! \nBasic mod that adds a few items, mobs, and blocks.
```

### Congrats!

You are finally done configuring everything!

To make sure that everything is working, run this command in your terminal:

```bash
./gradlew genIntellijRuns
```
{: .nolineno }

> You may get a ton of warnings; that is okay, we just want to make sure we still get the `BUILD SUCCESSFUL` message.
{: .prompt-warning }

This command will bring up many different things you can run in your **right side gradle tab**. We will use **runClient** to test our mod. Feel free to run this to make sure everything is working.

Now, it is time to do some modding!

