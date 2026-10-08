---
title: "Routing, Renders and Loading State"
parent_post: React-FocusTube
module_number: 6
layout: module
media_subpath: /assets/tutorials/focustube
---

## React vs. React Native: Routing & Renders

If you have built websites using React or NextJS, you might be wondering about some of the differences:

- **No SSR (Server Side Rendering):** In NextJS, components run on the server by default. React Native runs **entirely on the client device**. There is no server rendering, which means we do not need to write `'use client'` at the top of our files!
- **Unified Hooks:** Since everything is on the client, you can use React hooks like `useState` and `useEffect` anywhere in your application without restrictions or special compiler flags.
- **`useLocalSearchParams` vs `useParams`:** NextJS uses `useParams` from `next/navigation` to read route params. Expo Router uses `useLocalSearchParams` to read route parameters on mobile devices.

## Loading State

When loading data in a mobile app, it is important to give the user visual feedback so they know the app hasn't crashed.

While NextJS has a special `loading.js` file convention for server-side loading boundaries, React Native apps typically handle loading state using standard React `useState` variables and Native loading indicators like the **`<ActivityIndicator>`** component from `react-native`.

Here is an example of how you can implement a loading screen in React Native:

```tsx
import React, { useState } from 'react';
import { ActivityIndicator, View, Text } from 'react-native';

export default function LoadingExample() {
  const [isLoading, setIsLoading] = useState(true);

  if (isLoading) {
    return (
      <View className="flex-1 bg-black justify-center items-center">
        <ActivityIndicator size="large" color="#ef4444" />
        <Text className="text-white mt-4">Loading content...</Text>
      </View>
    );
  }

  return (
    <View className="flex-1 bg-black justify-center items-center">
      <Text className="text-white">Content loaded!</Text>
    </View>
  );
}
```

## Completion & Discussion Checklist

Before joining the group discussion or moving on to the next module, ensure you have completed the tasks, investigated the bugs, and are ready to discuss the questions below:

<details markdown="1">
<summary>Click to expand Completion & Discussion Checklist (12 Items)</summary>

| # | Type | Item | Prompt Preview |
| :-: | :--- | :--- | :--- |
| 1 | Bug Hunt | YouTube Player Dimensions Bug | In the video component, required player dimensions were omitted. Check the Expo compiler logs to identify the missing properties and restore the video view. |
| 2 | Question | Babel Transpilation | Take a moment to explore what Babel is and how it transforms modern JSX and TypeScript into device-compatible JavaScript. |
| 3 | Question | Expo Configuration Files | What are `package.json` and `app.json` used for in Expo, and how does `app.json` control native app metadata? |
| 4 | Question | Component Definition | What is a React component? Learn a high-level definition to understand how components structure mobile interfaces. |
| 5 | Question | Component Return Values | What do React Native component functions return under the hood? |
| 6 | Question | Third-Party Package Trade-offs | What are the benefits and potential trade-offs of relying on third-party NPM packages in a mobile project? |
| 7 | Question | `<Link>` vs. Standard Buttons | Why do we import the `<Link>` component instead of just using a standard mobile button for screen transitions? |
| 8 | Question | `onChangeText` Event Model | How does React Native's `onChangeText` event differ from the standard web `onChange` event handler? |
| 9 | Question | `useState` UI Reconciliation | Why can't we use a simple `let input = ""` variable? What does `useState` do under the hood to trigger re-renders? |
| 10 | Question | Dynamic Segment Routing (`[videoId]`) | Why does the route filename use brackets `[videoId]`? What will the parameter value be when navigating to `/video/abc`? |
| 11 | Task | Playlist Screen Component | Inside `app/playlist/index.tsx`, create a React Native component displaying mock video cards styled with NativeWind linking to `/video`. |
| 12 | Challenge | Dynamic Video Param Binding | Connect `useLocalSearchParams()` in `app/video/[videoId].tsx` to dynamically pass the `videoId` to `<YoutubePlayer>`. |

</details>
