---
title: "Calling the API and Displaying the Data"
parent_post: React-FocusTube
module_number: 9
layout: module
media_subpath: /assets/tutorials/focustube
---

## Calling the API and Displaying the Data

Now that we have our service layer, we can load this data directly into our React Native screens! 

Let's work on our `/search` screen first.

### Fetching Data inside `useEffect`

Because fetching data from the internet takes time, we run it asynchronously. In React, we use the `useEffect` hook to trigger our API call when the screen loads, and use `useState` to save the results so the screen re-renders once the data arrives.

Here is the implementation of `app/search/[searchId].tsx`:

```tsx
import React, { useEffect, useState } from 'react';
import { View, Text, Image, ScrollView, ActivityIndicator, Pressable } from 'react-native';
import { Link, useLocalSearchParams } from 'expo-router';
import { searchVideos } from '../../services/youtube';

export default function SearchPage() {
 const { searchId } = useLocalSearchParams();
 const [results, setResults] = useState<any>(null);
 const [loading, setLoading] = useState(true);

 useEffect(() => {
   const getResults = async () => {
     try {
       // encodeURIComponent ensures weird characters don't break our URL
       const data = searchVideos(searchId as string, "video");
       setResults(data);
     } catch (error) {
       console.error(error);
     } finally {
       setLoading(false);
     }
   };

   getResults();
 }, [searchId]);

 if (loading) {
   return (
     <View className="flex-1 bg-black justify-center items-center">
       <ActivityIndicator size="large" color="#ef4444" />
     </View>
   );
 }

 return (
   <ScrollView className="flex-1 bg-black p-4">
     <Text className="text-white text-3xl font-bold mb-6">Search Results</Text>
     
     <View className="pb-10">
       {results?.items?.map((vid: any) => (
         <Link href={`/video/${vid.id.videoId}`} asChild key={vid.id.videoId}>
           <Pressable className="w-full bg-neutral-800 rounded-xl p-3 flex-row mb-4">
             <Image 
               source={{ uri: vid.snippet.thumbnails.medium?.url }}
               className="w-40 h-24 rounded-lg"
             />
             <View className="flex-1 ml-3 justify-start">
               <Text className="text-white text-base font-bold" numberOfLines={2}>
                 {vid.snippet.title}
               </Text>
               <Text className="text-gray-400 text-xs mt-1" numberOfLines={2}>
                 {vid.snippet.description}
               </Text>
             </View>
           </Pressable>
         </Link>
       ))}
     </View>
   </ScrollView>
 );
}
```
{: file="app/search/[searchId].tsx" }
{: .nolineno }

> **BUG HUNT:** The code above compiles perfectly without crashing, but when you run it, the screen stays blank forever (the loading spinner disappears, but no search results show up). Look closely at the `useEffect` hook. Can you find what's missing when we call the asynchronous API function?
{: .prompt-danger }

> **QUESTION:** In the code above, we use the `numberOfLines` property on the `<Text>` components. Why is this important when rendering dynamic data from an external API on mobile devices?
{: .prompt-tip }

### Video Screen Integration

Now that clicking a video card links to `/video/[videoId]`, let's update `app/video/[videoId].tsx` to load the actual YouTube Player and display the video details!

```tsx
import React, { useEffect, useState } from 'react';
import { View, Text, ScrollView, ActivityIndicator } from 'react-native';
import { useLocalSearchParams } from 'expo-router';
import YoutubePlayer from 'react-native-youtube-iframe';
import { fetchVideoDetails } from '../../services/youtube';

export default function VideoScreen() {
  const { videoId } = useLocalSearchParams();
  const [video, setVideo] = useState<any>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const getDetails = async () => {
      try {
        const data = await fetchVideoDetails(videoId as string);
        setVideo(data.items?.[0] ?? null);
      } catch (error) {
        console.error(error);
      } finally {
        setLoading(false);
      }
    };

    getDetails();
  }, [videoId]);

  if (loading) {
    return (
      <View className="flex-1 bg-black justify-center items-center">
        <ActivityIndicator size="large" color="#ef4444" />
      </View>
    );
  }

  const title = video?.snippet?.title ?? "No Title";
  const description = video?.snippet?.description ?? "No Description";

  return (
    <ScrollView className="flex-1 bg-black">
      <View className="p-4">
        <YoutubePlayer
          height={220}
          play={false}
          videoId={videoId as string}
        />
        <Text className="text-white text-2xl font-bold mt-4">{title}</Text>
        
        <View className="bg-neutral-800 p-4 rounded-xl mt-4">
          <Text className="text-gray-300 text-sm leading-relaxed">{description}</Text>
        </View>
      </View>
    </ScrollView>
  );
}
```
{: file="app/video/[videoId].tsx" }
{: .nolineno }

> **QUESTION:** Look at how we fall back to `"No Title"` or `"No Description"` using the `??` operator. What is this operator called in JavaScript, and how does it prevent our app from crashing if the YouTube API returns null values?
{: .prompt-tip }

### Challenge Task: Build the Playlist Page

> **TASK: Complete the Playlist Screen**
> Give an attempt at making the playlist page inside `app/playlist/[playlistId].tsx`.
> - Import the `fetchPlaylistVideos` function from your API service.
> - Fetch the playlist items using the `playlistId` from the route parameters.
> - Display them in a list of video cards similar to your search screen.
> - Make sure each video card successfully links to the video screen!
{: .prompt-warning }
