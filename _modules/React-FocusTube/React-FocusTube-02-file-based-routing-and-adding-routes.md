---
title: "File-Based Routing and Adding Routes"
parent_post: React-FocusTube
module_number: 2
layout: module
media_subpath: /assets/tutorials/focustube
---

## File-Based Routing

**File-Based Routing** means that the structure of your files and folders inside the **app/** directory *automatically* defines the screens in your mobile app.

Let’s take a look at what this really means.

In your **/app** folder create a folder called *```video```*. In this folder create a file called **index.tsx**. In this file, put something really simple like this:

> **NOTE:** In Expo Router, folders wrapped in parentheses like `(tabs)` are **Route Groups** used to organize shared layouts (like tab navigation) without adding the group name to the URL path. Standard folders without parentheses (like `video`) map directly to the URL route (e.g. `/video`). Since we want a standalone `/video` route, we create a normal folder named `video` directly inside `app/` (outside the `(tabs)` folder).
{: .prompt-info }

```tsx
import { Text, View } from 'react-native';

export default function Video() {
   return (
     <View className="flex-1 items-center justify-center bg-black">
       <Text className="text-white text-2xl">This is the Video page!</Text>
     </View>
   );
}
```
{: file="app/video/index.tsx" }
{: .nolineno }

Next, how do we see it? In Expo Router, you can navigate by altering the URL or pushing a route. If you are using Expo Go on your phone, you don't have a URL bar. But don't worry, we will add navigation buttons shortly!

> **QUESTION:** What is a component? They are key to truly mastering the power of React, so learn a high-level definition to keep in the back of your mind since this term will come up again and again.
{: .prompt-tip }

## Adding more Routes

Now that we know what file-based routing is, make **2 more routes**.

- ```/search```
- ```/playlist```

For now, these can display whatever you like.

Here is what your file tree should look like now:

```
my-app/
├── app/
│   ├── (tabs)/
│   │   ├── _layout.tsx
│   │   ├── explore.tsx
│   │   └── index.tsx
│   ├── playlist/
│   │   └── index.tsx
│   ├── search/
│   │   └── index.tsx
│   ├── video/
│   │   └── index.tsx
│   ├── _layout.tsx
│   └── modal.tsx
```

You should have **4 routes** in your app: the home screen (`index.tsx`), ```/search```, ```/playlist```, and ```/video```.

Now that we have a couple routes, we can start to change how they look. Feel free to use the instructions as a guide to your own design, or copy it if you already feel comfortable in React and JS.

### The Power of NPM Packages (Video Route)

It is pretty easy to add YouTube videos into your app. If we were building a website, we would just use an HTML `<iframe>`. However, React Native compiles to actual mobile code, which doesn't support HTML tags!

Instead of writing complex Native iOS and Android code from scratch to display a video, we can use an **NPM Package**. Packages are pre-written blocks of code created by the community that we can download and use in our app instantly.

Run this command to install a popular YouTube video package:

```console
npm install react-native-youtube-iframe react-native-webview
```

> **QUESTION:** What are the benefits of using third-party packages instead of writing everything from scratch? Are there any potential downsides?
{: .prompt-tip }

Now, we can import this package and use it in our `/video` route:

```tsx
import { View } from 'react-native';
import YoutubePlayer from "react-native-youtube-iframe";

export default function Content() {
 return (
   <View className="flex-1 bg-black justify-center">
     <YoutubePlayer
       play={false}
       videoId={"your-youtube-video-id-here"}
     />
   </View>
 );
}
```
{: file="app/video/index.tsx" }
{: .nolineno }

> **BUG:** In the above answer I purposely **omitted one of the required properties** of `YoutubePlayer`. See if you can spot what is missing or check your terminal logs!
{: .prompt-danger }

### Search Route

Next up, we have our `/search` route.

> **NOTE:** This may sound counter-intuitive, but the search page will display the search results, not the actual search bar. The search bar will be added on the *home page*, which we will get to shortly.
{: .prompt-info }

First, let's create a basic page component that just returns a title.

```tsx
import { View, Text } from 'react-native';

export default function SearchPage() {
 return (
   <View className="flex-1 bg-black items-center pt-10">
     <Text className="text-white text-3xl font-bold">Search Results</Text>
     {/* Video cards will go here */}
   </View>
 );
}
```
{: file="app/search/index.tsx" }
{: .nolineno }

> **QUESTION:** This is the second component we have seen. You may start to pick up a pattern on how they are constructed. What do these component functions return?
{: .prompt-tip }

Next, we need a way to mock a single "Video Card". A video card should have a thumbnail image, a title, and a description. We also want the entire card to be clickable so it can take the user to the `/video` route we just made! A perfect job for the Expo `<Link>` component.

Let's import `<Link>` from `expo-router` and `<Image>` from `react-native`, then add a dummy video card:

```tsx
import { View, Text, Image, Pressable } from 'react-native';
import { Link } from 'expo-router';

export default function SearchPage() {
 return (
   <View className="flex-1 bg-black items-center pt-10 px-4">
     <Text className="text-white text-3xl font-bold mb-6">Search Results</Text>
     
     <Link href="/video" asChild>
       <Pressable className="w-full bg-neutral-800 rounded-xl p-3 flex-row mb-4">
         <Image 
           source={{ uri: "https://i.ytimg.com/vi/abc123/mqdefault.jpg" }}
           className="w-40 h-24 rounded-lg"
         />
         <View className="flex-1 ml-3 justify-start">
           <Text className="text-white text-lg font-bold">How to Learn JavaScript</Text>
           <Text className="text-gray-400 text-sm mt-1">A quick guide to getting started with JavaScript.</Text>
         </View>
       </Pressable>
     </Link>

   </View>
 );
}
```
{: file="app/search/index.tsx" }
{: .nolineno }

Wait, what is a `<Link>` component? Using this prebuilt component gives us a really cool ability...

> **QUESTION:**  Why do we import the `<Link>` component instead of just using a standard mobile button? 
{: .prompt-tip }

You can copy and paste the `<Link>` block multiple times if you want to see what a list of results looks like. Later in the tutorial, we will replace this hardcoded fake data with real API calls!

### Playlist Route

> **TASK:** Make the playlist route. It is very similar to the search route. You got this!
{: .prompt-warning }

- Inside ```app/playlist/index.tsx```, create a React Native component that shows fake video cards in a playlist.
- Style each video as you want using NativeWind classes.
- Make sure each one links to the ```/video``` route!
