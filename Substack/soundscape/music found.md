This article is a introduction to the player concept
- LIVE

# Music Found
### A Player for the Music I Love
date: Oct 02. 2025

I miss my Zune player. Today, music is all about streaming, but platforms like Spotify, Bandcamp, and SoundCloud all miss something vital: the personal connection to music. Musicians have a vision and still love to provide a package deal, rich visuals, art, and stories on the cover or sleeve. Why has no one brought that experience to the web? That's why I've focused on building a great player for the music you already have, with the hope of someday adding purchasing power so musicians can sell their products directly, like Bandcamp, or artist, like Deviant Art.

Thanks for reading Developer3027! Subscribe for free to receive new posts and support my work.

Even if you have your own music, how will you play it today? Spotify, Soundcloud, Audius, Bandcamp, Pandora all have good points and bad, but none allow you to play your music. You have a cd, album, cassette, then you need equipment. If you have music, then you can play it isolated, or you can stream. I am sure there is a web player out there, but I don’t know what it is. I want to fully customize my music experience. Make it mine, custom to my experience. My experience is likely, not like yours.

Take the song “Every Breath You Take” by the Police. It is a very popular song for wedding playlists, but the song is about obsessive jealousy, a stalker, and the feeling of being watched by a "Big Brother" figure, according to Sting. You may have heard it at a wedding and want to cherish that memory. Why not have a live image of the bride and groom dancing when playing it. Or you may want a image of Mr Hyde. Either way, I want that option. I want to make it mine.

Thanks for reading Developer3027! This post is public so feel free to share it.

Share

I have started building out a web app called Music Found in Rails to solve this problem. In the Rogue Media Lab site it is called Zuke. It has a long way to go for sure. This project has been a learning journey for me, and in these articles, I'll share how I'm building it from the ground up using technologies like Rails, Stimulus, Turbo Frames and the wavesurfer.js library. I am vibe coding this project with Kilo Code in VSCode as well as Gemini.

I am really enjoying vibe coding as it has allowed me to create and experiment quickly as well as have a mentor that knows a fair bit about what it is doing. It has been really nice to get answers to why this works or why this way was preferred over that way.

If you did not know, wavesurfer.js is a javascript library used to create customizable and interactive waveforms. It has a ton of functionality built in and I use it for the audio heavy lifting. With Stimulus in Rails, this is a breeze.

Waveurfer.js may handle the audio, but the player I created has a few different parts that all work together. There is the banner, which is a image that sits on top of the waveform. This banner can show a artist image or a short video. The waveform, which I have designed to feel like Soundcloud’s. A loading bar and finally the control bar that includes the play, pause, previous and next buttons.


Leave a comment

This player is driven through creating and destroying custom events in the browser called the Global Event Bus. With this set up I can have the player load and play the song as well as show the banner image. Each component dealing with it’s individual part.

I hope you find this as fun as I have. This project has taught me so much about web audio and building apps. If you've ever felt this way about your music, join me on this journey as we build an app that puts the control back in your hands. Next week, we'll dive into how the core music player works.