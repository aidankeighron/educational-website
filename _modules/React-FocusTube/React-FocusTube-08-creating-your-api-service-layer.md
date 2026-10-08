---
title: "Creating Your API Service Layer"
parent_post: React-FocusTube
module_number: 8
layout: module
media_subpath: /assets/tutorials/focustube
---

## Creating your API Service Layer

Since React Native does not run on a server, we do not build `/api/...` server routes like you would in NextJS. Instead, we make HTTP requests directly from our app.

However, writing direct `fetch()` calls inside our visual UI components is bad practice. It makes the code cluttered and hard to maintain. To keep our code clean, we will create a dedicated **API Service Layer** to abstract these details.

In your project root, create a new folder named `services/`. Inside it, create a file named `youtube.ts`.

We need three different functions in this service to handle our data:
- `fetchVideoDetails(videoId)`
- `searchVideos(query, type)`
- `fetchPlaylistVideos(playlistId)`

**This is what your tree structure should look like now**:

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
└── services/
    └── youtube.ts
```

### Writing the Service Logic

We will use the JavaScript `URL` helper class to dynamically build our query strings with query parameters (e.g. `?part=snippet&key=...`).

Here is the helper code for `services/youtube.ts`:

```typescript
const BASE_URL = "https://www.googleapis.com/youtube/v3";
const API_KEY = process.env.EXPO_PUBLIC_API_KEY;

export async function fetchVideoDetails(videoId: string) {
  const url = new URL(`${BASE_URL}/videos`);
  url.searchParams.set("part", "snippet,contentDetails,statistics");
  url.searchParams.set("id", videoId);
  url.searchParams.set("key", API_KEY || "");

  try {
    const response = await fetch(url.toString());
    if (!response.ok) {
      throw new Error("Failed to fetch video details");
    }
    return await response.json();
  } catch (error) {
    console.error("fetchVideoDetails Error:", error);
    throw error;
  }
}

export async function searchVideos(query: string, type: string) {
  const url = new URL(`${BASE_URL}/search`);
  url.searchParams.set("part", "snippet");
  url.searchParams.set("q", query);
  url.searchParams.set("type", type);
  url.searchParams.set("key", API_KEY || "");
  
  if (type === "video") {
    url.searchParams.set("videoDuration", "medium");
  }

  try {
    const response = await fetch(url.toString());
    if (!response.ok) {
      throw new Error("Failed to search videos");
    }
    return await response.json();
  } catch (error) {
    console.error("searchVideos Error:", error);
    throw error;
  }
}

export async function fetchPlaylistVideos(playlistId: string) {
  const url = new URL(`${BASE_URL}/playlistItems`);
  url.searchParams.set("part", "snippet");
  url.searchParams.set("playlistId", playlistId);
  url.searchParams.set("maxResults", "25");
  url.searchParams.set("key", API_KEY || "");

  try {
    const response = await fetch(url.toString());
    if (!response.ok) {
      throw new Error("Failed to fetch playlist");
    }
    return await response.json();
  } catch (error) {
    console.error("fetchPlaylistVideos Error:", error);
    throw error;
  }
}
```
{: file="services/youtube.ts" }
{: .nolineno }

> **QUESTION:** Why do we wrap our API requests in `try/catch` blocks? What kinds of real-world errors (like network loss or API rate limits) could cause the `catch` block to execute?
{: .prompt-tip }
