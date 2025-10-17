Current State

I just set this folder up this morning and have been working to populate it with information for you. I want to utilize you and your skills to help me with this studio. I work off this laptop but much of my content is scattered in various folders or online. This presented a challenge with setting things up for you. Please bare with me while I get better organized for you.

1. Portfolio:
- This will be the official "Rogue Media Lab" site.
- The projects within the site should feel like actual sites, but stay inside RML. These are a reflection of the actual projects being built and provide the flavor, not the substance of the actual project.
- Salt and Tar is actually a part of RML Voyager. The Voyager project will include many sailing content providers. I used Salt and Tar because I love that design and had fun with features. Design in Figma.
- Hermit Plus is a big favorite of mine that I can never find the time for. This landing page is actually love on AWS via a old account I no longer have access to. A big part of this concept is built in NextJS which I have the code for. The design is in Figma.
- Zuke is actually what the Soundscape music player has grown from.
- At the end of the day, these sub projects are just a showcase of what the studio can do and reflections of working projects.
- The original redesign for the portfolio had a "Fallout" screen vibe and feel. I like the modern flow and feel of Patreon if I am honest, however the portfolio will include functionality on par with the studio needs. Patreon is a means to an end and support the studio until it can stand on it's own. Overall I think the site works. Needs some polish, not a rewrite.
- It has been live on Heroku using there PG database and worked wonderfully. Even with S3.

2. RML Soundscape:
- The code in the Soundscape folder was grabbed from the github repo main branch, which is not the most current code. It was more than enough to give you the concept so we can have this chat. The player currently works, in both the full project and the portfolio version. It has been live on Heroku and S3, all works well. The player is also good on mobile, just does not have it's own version built for ios or android. It also does not have a dedicated mobile or desktop version. However both desktop and mobile work as intended currently. You can upload a song, play the song, banner images and video work, waveform works and the controls work. Swipe gestures are fussy. Playlists need review as it is clunky. We will talk MVP once I get the full version over for you.

3. Deployment and hosting:
- No apps are currently live, but both have been on Heroku. I have also used Digital Ocean.
- Both apps my use the same S3 bucket currently? If so that can be addressed when we are ready. I do not see that as a development concern but rather a deployment one. I could be wrong.

4. Community and Content:
- Patreon is launched. Current count is 0.
- 19 articles published on Substack. Turned on the paid subscriber last week. Writing the first paid article now.
- At minimum, all the socials listed in the socials-info.md file are active and have at least 3 posts.

Strategic Questions

5. Immediate Priorities:
- I have no idea what needs to happen to attract the Patreon's or Substack. Guy I love on Substack called Nate uses TicTok to drive to his Substack.
- Soundscape is my flagship I would say. I love working on the app and love playing with it. I feel that it has great potential with the economy headed in the direction it is. People will want to make there old stuff new and hang on to it.

6. Missing Infrastructure:
- The portfolio site will be the central space for Rogue Media Lab. It will showcase the studio and so much more. 
- RML does need a domain. 
- The email is gmail, so works but not pro, and there is a contact system started in the portfolio.
- I will build analytics into the portfolio yes.

7. Content and Marketing
- Need to tighten up on all of this section.

8. Technical Gaps:
- Yes auth is complete for Soundscape and the Portfolio.
- Have not completed payments for Patreon. Is complete for Substack. I will add a stripe feature into the Portfolio.
- I think some api endpoints are built for both. I think it is a great idea for future proofing. There is probably room to improve here, if I am honest.

What's the ONE thing that, if we finished it today, would make you feel like RML just took a big step forward? Probably the biggest thing on my mind the past few days is the paid article for Substack. I have a version started but not 100 that the content is solid, the idea is solid, or that this is a good idea.