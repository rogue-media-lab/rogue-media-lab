This article is setting up the Rails app using templates and getting the music organized.
- LIVE

# Soundscape
### Starting the visual music player.
date: Oct 09, 2025

Web Development
This app is built with Rails 8, styled with Tailwind, and uses a PostgreSQL database. While I've touched on these topics before, for this series I want to start at the very beginning by walking you through my system setup on a Pop!_OS powered Thinkpad. You should have Rails, Ruby 3.3.7, and PostgreSQL 17.5 installed. We can then use my templates to spin up a new app and configure the database.

Rogue Media Lab is a reader-supported publication. To receive new posts and support my work, consider becoming a free or paid subscriber.

In general, when spinning up a new Rails app you will run this command in the console or terminal:

rails new app_name -d postgresql -c tailwind
Then you can start adding gems and configuring your new app. I have created a few templates that I use to help automate this and makes my life so much easier. To use a template you pass the -m flag and provide the location for the template. This could be a local or online address. It would look like this, adding to the example above:

rails new app_name -d postgresql -c tailwind -m /local_template_file.rb
Streamlining with templates
I have created some templates for Rails that will make setting up a new app quick and easy. I created a Github repo for them and there are different branches for different set ups. There are two I would highly recommend.

Rails, like most frameworks, is modular and installs packages called gems. These are contained bits of code that all work together to provide that Rails magic. Rails uses a package called bundle to install all these gems and puts them in a folder called bundle.

Much like the Node ecosystem, Rails uses a file called the gem file to list out all these gems, just like the package.json file does for Node. Bundle uses this file to install the gems needed for the app. This means that, like the Node ecosystem, you can grab the code from Github and run “bundle install” and bundle will do it’s thing. There is no need to push all the gem files up to Github.

The default Rails setup doesn't add the `vendor/bundle` folder to the `.gitignore` file, which is a problem because it contains all the code for the gems your app uses. This means that a fresh install will track thousands of unnecessary files. My template fixes this, ensuring your repository only tracks the code you write, which is the standard practice in a professional environment.

The main branch of the template repo is the template to fix this issue. It provides a new `.gitignore` that includes the ignore for the `vendor/bundle/*` folder as well as fixing the git workflow to run properly now that there are no files in the bundle folder. This drops the number of files to push on initial considerably.

The second template I recommend is the Tailwindcss template. This will set up tailwind styles for you, including giving you the custom themes template and a markdown file on how to use and modify it. This makes getting up a running with Tailwind very quick and easy with the newer rails version.

I would like to mention that I also have a template that will install flash messages, complete with a stimulus controller. This is just a simple set up so nothing fancy but it works out of the box and very customizable.

Now that I have the app installed I want to open it in VSCode. Open the `database.yml` file located in the config folder and add host, username, and password to the default. It should look like this:

default: &default
  adapter: postgresql
  encoding: unicode
  host: localhost
  username: postgres
  password: postgres
  # For details on connection pooling, see Rails configuration guide
  # https://guides.rubyonrails.org/configuring.html#database-pooling
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
Now you can create the database and run your dev server. Now that we have the app installed, let’s talk about how to get the most out of it. We can start by organizing our music.

Organize your music collection
You may also want to go ahead and get your music organized. The way I set the player up was to focus on the song. Most every song is going to be part of a album and every song will have a artist, but initially, I want to play a song and have a image for it.

So I went through my music folder, ensuring that every song was in an artist folder. In the artist folder I had a album folder and in that folder there were the songs for the album. Some song may be singles and in that case they were put in the artist folder. Every artist has a image. Every album has a image.

music folder
music folder
music folder
Music Folder
Like I mentioned before, I want this music to be mine, as I would enjoy it. I spent time looking through Deviant art, finding images that best connected with how the music made me feel. In other instances I found great images of the band or artist. I tried to keep album art to the original concept.

For this app, when clicking on a song, the banner image will change to the artist image, or video. There are any number of ways to handle this, but I decided to start with the song model, then deal with the other data (like artist album, genre ), and show the artist image as the banner. This image is 1280 x 300 for this build. If you are including a video, the dimensions for it should be the same.

To ensure a clear and consistent flow in the app, I established a straightforward data hierarchy: every Song belongs to an Album, which in turn belongs to an Artist. This structure allows me to easily associate the song with the artist's visual assets, like their banner image or video. When you play a song, the app can then display the associated artist's banner.

Join Mason Roberts’s subscriber chat
Available in the Substack app and on web
Now that our foundation is set, it's time to bring our vision to life. Next week, we'll take our first steps toward building a player that truly makes your music feel alive. We’ll get our hands dirty with some code, seed our first songs, and see the Wavesurfer.js waveform come alive.