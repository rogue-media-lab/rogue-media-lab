First paid article for the Soundscape series. This will focus on why I refactored the player. Separation of Concerns and managing expectations for this article.
- DRAFT

# Breaking down the Monolith
### Why I Rebuilt the Soundscape Player Before It Was Even Finished
date: Oct 16, 2025

When I first started the RML Soundscape music player, I was excited to get the player working. Every step, every bit of code, every browser refresh was a very exciting time. I had a vague design but I knew what I wanted and was building for that dream.

Rails is awesome that way. If you can dream it, you can build it. The player was visual so it needed a banner image. I love Soundcloud’s waveform so I needed to create that. Of course it needed controls. Finally it needed to play a song. This was all straight forward and that was what I tackled.

The view was broken into two parts. The top would be the player and the bottom would be the song cards. The idea was to click a card and the music would play. In playing the song the banner image would change. The waveform would track the song progression.

With this concept I made a partial view for the player and built the banner, waveform, and controls all in that view. It had a stimulus controller that handled everything in the music player partial. In the beginning that was fine as I was working on the MVP we just laid out.

When it finally all came together and it worked, I was so excited. It took about a week before I even realized there was a real problem. I had been so busy clicking and watching the banner change. I never let a song finish playing. I would click, then tweak or bug fix. Rinse and repeat. Loved it, but that day I let the song play from start to finish, I noticed a issue. That is what this article is about. Separation of concerns, and managing expectations.

As one may have expected, once the song was done, it did not play the next. I never wrote that code. This got me to thinking though. I wanted it to move on to the next, although this was not a playlist. If there was another song, I wanted it to play it. However, just playing the one song and being done, was not a bad thing. This was a option I did not want to get rid of.

This realization, that I needed options instead of a single, hard coded behavior, was the moment the original monolithic design was doomed. I needed components. The solution wasn’t just to add an auto play feature, it was to completely rethink the player’s architecture from the ground up.

For my paid subscribers, I’m going to walk you through the exact “Lego Brick Philosophy” I used to solve this problem. We’ll look at the “before” and “after” code and see how creating independent components makes the entire application more flexible, powerful, and easier to build upon.

--- Paywall ---

Like I said earlier, the original player was everything in one. The player was the banner, waveform, and controls. If I wanted to make changes, it was changing the player itself, not just a piece. When it came time to add that auto play option I realized that the player needed to evolve.

I had been thinking about the mobile player. It needed to be different than the desktop version. With the desktop and laptop, you had screen real estate. On the phone, I needed the banner to be prominent with the waveform on top. I may not need controls with swipes and gestures. That original build would not allow for this.

There were also times when the banner may not change, or the song would stick but the banner would change. This was because there were no real separations of concerns in the code. The entire controller ran the banner, song, waveform and controls. It was the code equivalent to food stuck in your teeth.

Here is what the initial player looked like:

```erb
<div data-controller=”music--player” class=”fixed flex justify-center items-center bottom-0 left-0 right-0 bg-black text-white p-4”>
  <div class=”flex flex-col”>
    <div data-music--player-target=”nowPlaying”>No song selected</div>
    <div data-music--player-target=”artistName”>No artist selected</div>
  </div>
  <!-- Current time -->
  <div data-music--player-target=”currentTime” class=”mx-2”>0:00</div>
  <div class=”flex flex-col w-1/2”>
    <div data-music--player-target=”waveform” class=”mx-2 px-2”></div>
    <div data-music--player-target=”loadingContainer” class=”hidden h-1 bg-gray-700 relative”>
      <div data-music--player-target=”loadingProgress” 
          class=”absolute top-0 left-0 h-full bg-lime-500 transition-all duration-300”
          style=”width: 0%”></div>
    </div>
  </div>
  <!-- Duration (updates when loaded) -->
  <div data-music--player-target=”duration” class=”mx-2”>0:00</div>
  <button data-music--player-target=”playerPlayButton”
          data-action=”click->music--player#togglePlayback”>
    <svg class=”w-10 h-10 text-gray-800 dark:text-white” aria-hidden=”true” xmlns=”http://www.w3.org/2000/svg” width=”24” height=”24” fill=”currentColor” viewBox=”0 0 24 24” data-music-target=”playIcon”>
      <path fill-rule=”evenodd” d=”M8.6 5.2A1 1 0 0 0 7 6v12a1 1 0 0 0 1.6.8l8-6a1 1 0 0 0 0-1.6l-8-6Z” clip-rule=”evenodd”/>
    </svg>
  </button>
  
  <button data-music--player-target=”playerPauseButton”
          class=”hidden”
          data-action=”click->music--player#togglePlayback”>
    <svg class=”w-10 h-10 text-gray-800 dark:text-white” aria-hidden=”true” xmlns=”http://www.w3.org/2000/svg” width=”24” height=”24” fill=”currentColor” viewBox=”0 0 24 24” data-music-target=”pauseIcon”>
      <path fill-rule=”evenodd” d=”M8 5a2 2 0 0 0-2 2v10a2 2 0 0 0 2 2h1a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2H8Zm7 0a2 2 0 0 0-2 2v10a2 2 0 0 0 2 2h1a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2h-1Z” clip-rule=”evenodd”/>
    </svg>
  </button>
</div>
```

You can see that the player is one solid chunk of code. This is early so the banner image is not included just yet, still, you can see how this is not really maintainable. Technically all those pieces with the data elements should be components.

In Rails there is a gem that helps create components. I tend to prefer just using partials and helpers when needed.


