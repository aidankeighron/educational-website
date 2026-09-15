---
title: "Final Challenge and Where to go from here"
parent_post: FocusTube
module_number: 11
layout: module
media_subpath: /assets/tutorials/focustube
---

## Final Challenge Task

Lastly, we need our `/search/[searchId]/page.js`:

- Call your playlist API to retreieve the playlist
- Go through the items (videos) in the search and display them as cards
- Be aware of edge cases! Find a way to not display shorts or any channels. (Yes, even if you specify "videos", they sometimes show up)
- Each card should link to the `/video/[videoId]/page.js` of that video

Your solution will be very similar to the playlist page. Utilize anything in this tutorial, Google, and documentation to help you.

## Completion & Discussion Checklist

Before joining the group discussion or concluding this tutorial, ensure you have completed the tasks, investigated the bugs, and are ready to discuss the questions below:

<details markdown="1">
<summary>Click to expand Completion & Discussion Checklist (9 Items)</summary>

| # | Type | Item | Prompt Preview |
| :-: | :--- | :--- | :--- |
| 1 | Bug Hunt | Search Route Partial Solution | The search route partial solution includes deliberate syntax and logic errors. Trace the request handler, fix the bugs, and verify it returns parsed video data. |
| 2 | Question | JSON vs. JavaScript Objects | How is JSON similar to a JavaScript object? Write a quick snippet showing how you would access a nested property from a fetched JSON payload. |
| 3 | Question | Query Parameter Design | What other information could an API accept through query parameters? Think of other platforms and what search filters they pass to backends. |
| 4 | Question | HTTP Status Code Conventions | Status codes tell the client what happened (e.g. `404` vs `500`). Why are status codes important, and what common status codes should you handle? |
| 5 | Question | YouTube Search Parameters | Looking at the search parameters (`part`, `q`, `type`, `videoDuration`), what does each represent when requesting data from the YouTube API? |
| 6 | Question | URI Parameter Encoding | What does `encodeURIComponent()` do to special characters in a search query, and why is it needed when constructing URLs? |
| 7 | Task | Create Playlist API Endpoint | Use the partial solution as a guide to create the `/api/playlist/route.js` endpoint using YouTube's `playlistItems` resource. |
| 8 | Task | Build Playlist View Page | Create `/app/playlist/[playlistId]/page.js`. Fetch playlist data from your endpoint and display responsive cards linking to `/video`. |
| 9 | Challenge | Search Page & Shorts Filtering | Build `/app/search/[searchId]/page.js` to display search results. Add filtering logic to exclude YouTube Shorts and channels. |

</details>

## **Extra**: Where to go from here?

### Congrats!

You have finished the main part of this tutorial!

There is a ton more you can do with this project, but you should have all the resources you need now to take it from here.

Here is a limited list of things you may consider for improving your website:
- Utilize the button we did not program yet (search for playlist)
- Edit "search" to accomodate playlists, or you can make a seperate playlist search (second would be easier, first is more ideal in OOD)
- Make a way to go through playlist videos
- Make a channel page
- Make a channel search page
- Show comments
- Show related videos
- An ambitious one could be to connect to your youtube account / use OAuth to view your own private content and playlists.
- Improving the design and overall look of the website

Overall, you should do whatever you want to make this project your own. Be creative and take it where you want it to go. If you thought of a different way to improve the website, do it! If you want to add something you did not see in this tutorial, don't be afraid to do it; use your resources and make it happen!

Thank you for sticking to the end. I hope you enjoyed the ride and learned something new!
