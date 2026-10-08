---
title: "Dynamic Routing and Using Params"
parent_post: React-FocusTube
module_number: 5
layout: module
media_subpath: /assets/tutorials/focustube
---

## Dynamic Routing

For many mobile apps, it is impossible to hardcode every single screen. This is one such case, as it needs to be able to render **any video on YouTube**.

Luckily, Expo Router has a pretty simple way to make the screen differ based on the parameters passed in the route. It's called **Dynamic Routing**.

**Here is an example:**

I have this route in my app:

`app/video/index.tsx`

Currently, this is kind of useless because there is only one video. However, with dynamic routing, we can insert the Video ID into the route to affect the screen.

In order to implement this, you will need to rename your route file to use brackets. It should look like this:

`app/video/[videoId].tsx`

Expo Router knows you have a dynamic route when the file or folder name has brackets around them.

> **QUESTION:** Why does the filename use brackets `[videoId]`? In a dynamic route, if a user navigates to `/video/abc`, what will the value of the parameter be?
{: .prompt-tip }

**Pay attention to what you name the file inside the brackets. That is what we will need to call in the next section.**

### Where we need Dynamic Routing

We need dynamic routing pretty much anywhere the screen is impacted by parameters, which happens to be **all of our routes** in this case.

So, you should:
- Rename your route files to use brackets *(/search, /video, /playlist)*
- Name your files whatever you want, they just have to be in brackets to work: `[name].tsx`

Your file tree should now look like this:

```
my-app/
├── app/
│   ├── (tabs)/
│   │   ├── _layout.tsx
│   │   ├── explore.tsx
│   │   └── index.tsx
│   ├── playlist/
│   │   └── [playlistId].tsx
│   ├── search/
│   │   └── [searchId].tsx
│   ├── video/
│   │   └── [videoId].tsx
│   ├── _layout.tsx
│   └── modal.tsx
```

#### Now navigate to each route by pushing `/route/[anything you want here]` (e.g. `/video/dQw4w9WgXcQ`)

Also, notice **if you try to navigate to just `/video`, it won't load the content page anymore.** This is because `index.tsx` was renamed into the dynamic route `[videoId].tsx`, **so a static `/video` route can no longer be found.**

## Using Params

The biggest reason why we use these dynamic routes is it makes it easy to pass information to the screen via the route URL.

In order to do so, we use the `useLocalSearchParams` hook from Expo Router.

- `useLocalSearchParams` is a React Hook that returns an object containing the dynamic route parameters.

```tsx
import { useLocalSearchParams } from 'expo-router';
import { Text, View } from 'react-native';

export default function Route() {
   // Destructure the parameter name you chose for your file.
   // If your file is [videoId].tsx, you destructure 'videoId'.
   const { videoId } = useLocalSearchParams();

   return (
     <View className="flex-1 bg-black justify-center items-center">
       <Text className="text-white"> This is a Route containing Video ID: {videoId} </Text>
     </View>
   );
}
```
{: file="app/video/[videoId].tsx" }
{: .nolineno }

### Implementing Dynamic Route Parameters

Since all of our routes are dynamic, in every route, **call `useLocalSearchParams()` and integrate it in some way to your component**, even if it's just displaying the search term or playlist ID on screen.

### Challenge Task

**Make the parameter in `app/video/[videoId].tsx` determine the video ID that plays in your YouTube Player!**

> **HINT:** Look at how the `<YoutubePlayer>` component takes its `videoId` property.
{: .prompt-tip }
