---
title: "Main and Layout Pages"
parent_post: React-FocusTube
module_number: 3
layout: module
media_subpath: /assets/tutorials/focustube
---

## Main and Layout Pages

### Main Page

Our main page will also be relatively simple. There are just a couple things to implement, all of which you should be able to do on your own. Working code will be included below, but make sure to attempt it yourself first.

**What the main page should look like:**
- A search input, in which you can type anything
   - Searching should not be possible when the input is blank
- 4 different buttons, each leading to 4 different types of searches
   - **Regular Search Button:** A regular video search
   - **Playlist Search Button:** A search for playlists specifically
   - **Video ID Button:** Instead of a search, if you know the video ID, type it here and it will go straight to the video
   - **Playlist ID Button:** Like the video ID button, this will take you directly to the playlist instead of searching for it

### Understanding React State (`useState`)

Before we build the search bar, we need a way to keep track of what the user is typing into it. In React Native, regular variables don't work for this because updating them doesn't tell the screen to re-render and show the new text. 

Instead, we use a React Hook called `useState`. It allows us to create a special variable (our state) and a function to update it. When we use the update function, React knows the state has changed and automatically updates our screen to reflect the new data!

Here is how we use it to track a text input in React Native:

```tsx
import { useState } from 'react';
import { TextInput } from 'react-native';

export default function MyComponent() {
  // We initialize our state variable 'text' to an empty string. 
  // 'setText' is the function we will call to update it.
  const [text, setText] = useState("");

  return (
    <TextInput 
      value={text} 
      onChangeText={(newText) => setText(newText)} 
    />
  );
}
```

> **QUESTION:** In React Native, we use `onChangeText` which provides the string directly. How does this differ from the web's `onChange` event?
{: .prompt-tip }

> **QUESTION:** Why can't we just use a standard local variable like `let input = ""` to track the user's input? What does `useState` do under the hood when its setter function is called?
{: .prompt-tip }

### Handling Button Clicks and Navigation

To make our navigation buttons work, we will also need the `onPress` event handler (the React Native equivalent of `onClick`). Let's see how that combines with navigation!

### Using Expo Router for Navigation

While `<Link>` is great for simple buttons and images, sometimes you need to navigate to a page after some logic runs—like when a user submits a search! 

For programmatic navigation, Expo Router provides the `useRouter` hook. 

```tsx
import { useRouter } from 'expo-router';
import { Pressable, Text } from 'react-native';

export default function MyComponent() {
  const router = useRouter();

  const handleAction = () => {
    // Sends you to another route in your app
    router.push('/specified/path');

    // Or go back in your navigation history
    // router.back();
  };
  
  return (
    <Pressable onPress={handleAction}>
       <Text>Go!</Text>
    </Pressable>
  )
}
```

**Click below to unblur the different parts of the answer**

**Part 1: State and Navigation Setup**
First, we need to set up our React state to keep track of what the user types into the search bar. We also initialize `useRouter()` so we can send the user to different pages when they click the buttons.
```tsx
import React, { useState } from "react";
import { View, Text, TextInput, Pressable } from 'react-native';
import { useRouter } from "expo-router";

export default function Home() {
 const [input, changeInput] = useState("");
 const router = useRouter();

 const handleSubmit = () => {
   if (input.trim() !== "") {
     router.push(`/search/${input}`);
   }
 };
 
 // ... return statement below ...
```
{: file="app/(tabs)/index.tsx (Part 1)" }
{: .nolineno }
{: .blur }

**Part 2: The Search Input**
Next, we build the search input. We tie the `<TextInput>` value to our state, and when the user submits their keyboard (triggering `onSubmitEditing`), we use `router.push()` to navigate to the search results!

Please don't skip through without giving it a solid attempt :)

```tsx
 // ... inside Home() return statement ...
 return (
   <View className="flex-1 bg-black items-center pt-20 px-4">
     <View className="mb-10 items-center">
       <Text className="text-white text-5xl font-bold">Focus</Text>
       <Text className="text-red-600 text-5xl font-bold">Tube</Text>
     </View>

     <View className="w-full flex-row items-center mb-6">
         <TextInput
           className="flex-1 bg-neutral-800 text-white p-4 rounded-l-lg text-lg"
           value={input}
           onChangeText={changeInput}
           placeholder="Search for something..."
           placeholderTextColor="#888"
           onSubmitEditing={handleSubmit}
         />
         <Pressable 
           className="bg-red-600 p-4 rounded-r-lg" 
           onPress={handleSubmit}
         >
           <Text className="text-white font-bold text-lg">Search</Text>
         </Pressable>
     </View>
     {/* ... navigation buttons below ... */}
```
{: file="app/(tabs)/index.tsx (Part 2)" }
{: .nolineno }
{: .blur }

**Part 3: The Navigation Buttons**
Finally, we add our quick-navigation buttons. Since these don't require submitting the keyboard, we can just use simple `onPress` events to check if the input is empty, and if not, route the user to the correct page.
```tsx
     {/* ... inside Home() return statement ... */}
     <View className="w-full flex-row flex-wrap justify-between">
       <Pressable
         className="bg-neutral-800 p-3 rounded-lg w-[48%] mb-4 items-center"
         onPress={() => {
           if (input.trim() !== "") router.push(`/search/${input}`);
         }}
       >
         <Text className="text-white font-semibold">Regular Search</Text>
       </Pressable>

       <Pressable
         className="bg-neutral-800 p-3 rounded-lg w-[48%] mb-4 items-center"
         onPress={() => {
           if (input.trim() !== "") router.push(`/playlist-search/${input}`);
         }}
       >
         <Text className="text-white font-semibold">Search Playlists</Text>
       </Pressable>

       <Pressable
         className="bg-neutral-800 p-3 rounded-lg w-[48%] mb-4 items-center"
         onPress={() => {
           if (input.trim() !== "") router.push(`/playlist/${input}`);
         }}
       >
         <Text className="text-white font-semibold">Search Playlist ID</Text>
       </Pressable>

       <Pressable
         className="bg-neutral-800 p-3 rounded-lg w-full items-center"
         onPress={() => {
           if (input.trim() !== "") router.push(`/video/${input}`);
         }}
       >
         <Text className="text-white font-semibold">Search Video ID</Text>
       </Pressable>
     </View>
   </View>
 );
}
```
{: file="app/(tabs)/index.tsx (Part 3)" }
{: .nolineno }
{: .blur }

### Layout Page (`_layout.tsx`)

This page is unique in Expo Router, but is also very powerful.

**The layout page is the UI that is shared between routes.**

In Expo Router, this file is called `_layout.tsx`. No matter where you are in the app, **what is in the layout page wraps your current screen**.

This often includes:

- Any global providers you want to add
- Headers and/or footers
- A Bottom Tab Navigation bar or Drawer Navigation

Feel free to make your Layout page your own. Luckily, Expo already has a really useful one made for you that you can tweak!
