---
title: "Final Challenge and Where to Go From Here"
parent_post: React-FocusTube
module_number: 10
layout: module
media_subpath: /assets/tutorials/focustube
---

## Final Challenge Task (Pagination & Filtering)

Lastly, YouTube's search endpoint sometimes returns channels or YouTube Shorts, even when we specify we want standard videos. 

- Filter your `.map()` or use a `.filter()` function to ensure we *only* display actual videos (`youtube#video`) and skip channels or playlists!

## Completion & Discussion Checklist

Before joining the group discussion or concluding this tutorial, ensure you have completed the tasks, investigated the bugs, and are ready to discuss the questions below:

<details markdown="1">
<summary>Click to expand Completion & Discussion Checklist (8 Items)</summary>

| # | Type | Item | Prompt Preview |
| :-: | :--- | :--- | :--- |
| 1 | Bug Hunt | Unresolved Promise Bug (Missing `await`) | TypeScript compiles without errors, yet no search results appear. Why does omitting `await` store an unresolved `Promise` in state, and how do we resolve it? |
| 2 | Question | JSON vs. JavaScript Objects | How is JSON similar to a JavaScript object? Write a quick snippet showing how you access nested properties from a fetched JSON payload. |
| 3 | Question | Mobile Client API Key Security | Why must Expo environment variables start with `EXPO_PUBLIC_`? What risks arise if private database secrets are bundled into client mobile binaries? |
| 4 | Question | Network Error Resilience | Why do we wrap API requests in `try/catch` blocks? What real-world mobile conditions (e.g. offline transitions, timeouts) trigger these blocks? |
| 5 | Question | Dynamic Text Layout Constraints | Why is setting `numberOfLines` on `<Text>` components essential when rendering dynamic external API content on small mobile screens? |
| 6 | Question | Nullish Coalescing (`??`) Operator | Look at how we fall back to `"No Title"` using the `??` operator. What is this operator called, and how does it prevent crashes compared to `||`? |
| 7 | Task | Playlist Screen Implementation | Build `app/playlist/[playlistId].tsx`. Fetch playlist items using `fetchPlaylistVideos` with `playlistId` from route parameters and render video cards. |
| 8 | Challenge | Video Filtering (Shorts & Channels) | Filter video results using `vid.id.kind === 'youtube#video'` to ensure only standard videos appear in the search feed. |

</details>

## Congrats!

You have finished the mobile migration of FocusTube!

There is a ton more you can do with this project. Here are some ideas:
- Utilize the button we did not program yet: Search Playlists.
- Make a dedicated Channel screen.
- Fetch and display comments below the video description.
- Connect OAuth authorization to display the student's *own* subscribed feeds and playlists directly!

Thank you for sticking to the end. I hope you enjoyed the ride and learned something new about mobile development with React Native and Expo!
