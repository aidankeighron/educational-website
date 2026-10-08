---
title: "Calling Custom API to the Front End to Display the Data"
parent_post: FocusTube
module_number: 10
layout: module
media_subpath: /assets/tutorials/focustube
---

## Calling Custom API to the Front End to Display the Data

Now that we actually have data to put in our website, we can now display everything.

First, we call the API in `/app/search`:

```jsx
 // encodeURIComponent is necessary since the searchId may contain
 // weird non unicode characters, this will fix that.
 const res = await fetch(
     `http://localhost:3000/api/search?text=${encodeURIComponent(searchId)}&type=video`,
     { cache: "no-store" } // this is optional
                           // included so there is always
                           // new / fresh data
 );
```

> **QUESTION:** What exactly does `encodeURIComponent()` do to a string like "lofi beats 2024!"? What potential issues could occur if we didn't use this function when passing user input into our URL?
{: .prompt-tip }

Just like how we called the YouTube API, we can call our own API. When we receieve the data, we can display it.

### How to Display API Data

> **Try this on your own**. Figure out what the API Data looks like and how you can display it. I will give a full solution to this one and a partial solution to another. You will need to do that last one on your own.
{: .prompt-warning }


Here is what an example video search that returns two videos would look like:

```json
{
 "kind": "youtube#searchListResponse",
 "etag": "dummyEtag123",
 "regionCode": "US",
 "pageInfo": {
   "totalResults": 2,
   "resultsPerPage": 2
 },
 "items": [
   {
     "kind": "youtube#searchResult",
     "etag": "etag1",
     "id": {
       "kind": "youtube#video",
       "videoId": "dQw4w9WgXcQ"
     },
     "snippet": {
       "publishedAt": "2023-01-01T00:00:00Z",
       "channelId": "UC123456789",
       "title": "Relaxing Lofi Beats",
       "description": "Perfect background music for studying and relaxing.",
       "thumbnails": {
         "default": {
           "url": "https://i.ytimg.com/vi/dQw4w9WgXcQ/default.jpg"
         }
       },
       "channelTitle": "Lofi Radio",
       "liveBroadcastContent": "none"
     }
   },
   {
     "kind": "youtube#searchResult",
     "etag": "etag2",
     "id": {
       "kind": "youtube#video",
       "videoId": "hY7m5jjJ9mM"
     },
     "snippet": {
       "publishedAt": "2023-01-02T00:00:00Z",
       "channelId": "UC987654321",
       "title": "Chillhop Essentials - Winter 2023",
       "description": "A selection of jazzy beats to relax or code to.",
       "thumbnails": {
         "default": {
           "url": "https://i.ytimg.com/vi/hY7m5jjJ9mM/default.jpg"
         }
       },
       "channelTitle": "Chillhop Music",
       "liveBroadcastContent": "none"
     }
   }
 ]
}
```

> If the above JSON is unreadable and confusing to you, do not worry. Paste any JSON object into [this website](https://jsoncrack.com/editor); it will help you visualize the data better.**
{: .prompt-tip }

Now that you know what the API data may look like, you can get to the data you need. For example:

- **Thumbnail** = `items[i].snippet.thumbnails.medium.url`
- **Title** = `items[i].snippet.title`
- **Description** = `items[i].snippet.description`
- **Video ID** = `items[i].id.videoId`

> **TIP:** If you need a refresher on how to extract information from JSON objects, here is a [quick demonstration](https://www.youtube.com/watch?v=iiADhChRriM).
{: .prompt-tip }


To show all the videos, we need to **iterate through all the videos in the items array and display a video card for each one**.

First, let's look at what a single video card would look like. We can use an anchor tag (`<a>`) to make the entire card clickable, wrapping the video's thumbnail, title, and description:

```jsx
<a href={`/video/${vid.id.videoId}`}>
    <img
        src={vid.snippet.thumbnails.medium.url}
        alt={vid.snippet.title}
        width={160}
        height={120}
    />
    <div>
        <h2> {vid.snippet.title} </h2>
        <p> {vid.snippet.description} </p>
    </div>
</a>
```

Now, instead of manually writing that block over and over for every single video, we can use JavaScript's `map()` function. This function goes through our array of videos one by one, and returns our card component for each video. 

Let's build it up:

```jsx
const videos = await res.json();

return (
  <div className="flex flex-col items-center w-full overflow-y-auto pb-10">
    {/* Map through the items array */}
    {videos.items?.map((vid) => { 
      
      // We return our anchor tag card for each 'vid'
      // Note: we can't use useRouter() since this is SSR, so we stick to <a>
      return (
        <a href={`/video/${vid.id.videoId}`} key={vid.id.videoId}>
          <img
              src={vid.snippet.thumbnails.medium.url}
              alt={vid.snippet.title}
              width={160}
              height={120}
          />
          <div>
              <h2> {vid.snippet.title} </h2>
              <p> {vid.snippet.description} </p>
          </div>
        </a>
      );
    })}
  </div>
);
```

Your full `/video/page.js` should look like this:

```jsx
"use client";

import { useParams } from "next/navigation";
import { useEffect, useMemo, useState } from "react";


export default function Content() {

 const { videoId } = useParams();

 // since this will be rendered later, since embed is an API
 // we can use useState() to set the video
 const [video, setVideo] = useState(null);

 // fetch the video and set it to the video variable
 // if there is one
 // review useEffect and useState if this seems confusing
 useEffect(() => {
   const fetchVideo = async () => {
     const res = await fetch(`/api/video?videoId=${encodeURIComponent(videoId)}`, { cache: "force-cache" });
     const vid = await res.json();
     setVideo(vid?.items?.[0] ?? null);
   };

   fetchVideo();
 }, [videoId]);

 // get the snippet from the video to extract
 const title = video?.snippet?.title ?? "No title";
 const description = video?.snippet?.description ?? "No description";

 // now display all the information
 return (
   <>
     <div className="h-lvh w-lvw flex flex-col items-center justify-start text-white">
       <h1 className="text-3xl p-2 m-2">{title}</h1>
       <iframe
         width="960"
         height="540"
         src={`https://www.youtube.com/embed/${videoId}`}
         title={title}
         frameBorder="0"
         allowFullScreen
       ></iframe>
       <div className="flex items-center justify-between max-h-1/4 w-full max-w-5xl p-4 mt-4 bg-neutral-700 rounded-lg overflow-scroll"> 
        <p className="mt-4 text-gray-300">{description}</p>
       </div>
     </div>
   </>
 );
}
```
{: file="app/video/[videoId]/page.js" }
{: .nolineno }
{: .blur }

> **TASK: Build the Playlist Page**
> 
> Please give an attempt at making the playlist page. You will need to:
> - Call your playlist API to retrieve the playlist
> - Go through the items (videos) in your playlist and display them as cards
> - Make sure each video links to the `/video/[videoId]/page.js` of that video
> 
> A partial solution with some intentional errors is given below, but given the demonstrations provided above, you should have more than enough to complete this on your own!
{: .prompt-warning }

```jsx
export default function PlaylistPage({ params }) {
  
   const { playlistId } =  params;

   const res = fetch(
       `http://localhost:3000/api/playlist?playlistId=playlistId`,
       { cache: 'no-store' }
   );

   const videos = res.json();

   return (
       <>
       <div className="w-screen h-screen flex flex-col items-center justify-start text-white">
           <h1 className="text-4xl mt-6 mb-4">Playlist Videos</h1>
           <div className="flex flex-col items-center w-full overflow-y-auto pb-10">
           {video.items.map((p) => {
               return (
               <a href={`/video/${p.snippet.resourceId.videoId}`} key={p.snippet.resourceId.videoId} className="w-1/2 max-w-3xl bg-neutral-700 rounded-2xl p-3 m-3 flex items-start gap-3 max-h-1/5">
                <img
                    src={p.snippet.thumbnails.medium?.url}
                    alt={p.snippet.title}
                    width={160}
                    height={120}
                    className="rounded-md shrink-0 object-fill w-40 h-30"
                />
                <div className="xflex flex-col justify-start h-full p-3">
                    <h2 className="text-lg font-semibold leading-tight mb-1"> {p.snippet.title} </h2>
                    <p className="text-sm text-gray-300 leading-snug max-h-3/4 overflow-scroll"> {p.snippet.description} </p>
                </div>
               </a>
           );
           })}
           </div>
       </div></>
   );
}
```
{: file="app/playlist/[playlistId]/page.js" }
{: .nolineno }
