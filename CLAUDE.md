# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

**Rogue Media Lab** is an indie development studio creating personality-driven media applications. This monorepo contains multiple Rails 8 applications focused on personal media ownership and creative expression. The founder Mason Roberts is an automotive technician building these projects as a "vibe coder" with a focus on transparent development and community engagement.

### Mission
Build creative applications that prioritize ownership over access, personality over polish, creation over consumption, and community over corporation.

## Project Structure

This is a monorepo containing:

- **Portfolio-App/milk-rails8-main**: MILK-00 portfolio showcase with multiple sub-projects
- **RML-Soundscape/music-found-main**: Music player for personally-owned music files
- **RML-Hermits**: Hermit Plus - Minecraft content hub (not yet fully developed)
- **RML-Voyager**: Additional project directory
- **Substack**: Content for Substack articles and documentation
- **Daily-Notes**: Development notes and progress tracking
- **Socials-Info**: Social media content and information

## Technology Stack

Both main applications use:
- **Ruby**: 3.3.7 (milk-rails8-main uses 3.3.0 in some areas)
- **Rails**: 8.x (Portfolio uses 8.0.2, Soundscape uses 8.0.1)
- **Database**: PostgreSQL with custom configuration
- **Styling**: Tailwind CSS
- **Frontend**: Hotwire (Turbo + Stimulus)
- **Storage**: AWS S3 for large files (audio, video, images)
- **Deployment**: Configured for Heroku and Kamal (Docker)

## Database Configuration

Both apps use PostgreSQL with non-standard defaults set in `config/database.yml`:
```yaml
host: "localhost"
username: "postgres"
password: "postgres"
```

### Database Commands

Portfolio App:
```bash
cd Portfolio-App/milk-rails8-main
bin/rails db:create
bin/rails db:migrate
bin/rails db:seed
```

RML Soundscape:
```bash
cd RML-Soundscape/music-found-main
bin/rails db:create
bin/rails db:migrate
bin/rails db:seed
```

## Development Workflow

### Starting the Development Server

Portfolio App:
```bash
cd Portfolio-App/milk-rails8-main
bin/dev
```

RML Soundscape:
```bash
cd RML-Soundscape/music-found-main
bin/dev
```

### Running Tests

```bash
bin/rails test
bin/rails test:system
```

### Code Quality

Both apps use:
```bash
bin/rails rubocop              # Lint Ruby code
bin/brakeman                   # Security scanning
```

### Asset Management

Tailwind CSS is compiled via:
```bash
bin/rails tailwindcss:build
bin/rails tailwindcss:watch    # For development
```

## Architecture Patterns

### Portfolio App (MILK-00)

**Multi-tenant structure** hosting several distinct sub-projects:
- **Salt and Tar**: Video archive for sailing YouTube channel with vintage aesthetic
- **Hermit Plus**: Minecraft content hub (landing page only)
- **Copywriter**: Portfolio concept for copywriting services
- **Blog**: Substack integration and content management
- **Zuke**: Early music player prototype (being replaced by RML Soundscape)

**Authentication**: Uses Devise with two separate models:
- `User`: Public user authentication
- `MilkAdmin`: Admin authentication (registrations disabled)

**Admin namespace**: All admin functionality under `milk_admin` namespace, with authenticated root routing.

**Key models**:
- `SaltAndTarVideo`: Video content with thumbnails, YouTube links, position ordering
- `Blog`, `BlogCategory`: Content management with friendly URLs
- `Project`, `Pill`: Portfolio items and skill tags
- `Song`, `Album`, `Artist`, `Genre`: Music metadata (legacy from Zuke)
- `HermitVideo`, `Hermit`: Minecraft content (early development)

**Routing pattern**: Clean URLs using dashes (e.g., `/salt-and-tar/archive`)

**Storage**: Active Storage with S3 backend for videos and images

### RML Soundscape (Music Found)

**Purpose**: Personal music player for owned audio files with visual customization.

**Core Philosophy**:
- You own the files (mp3s, etc.)
- Upload and manage your own music
- Customize visuals (images, short videos, metadata)
- Hierarchical data: Song → Album → Artist

**Music Player Architecture**:
- **WaveSurfer.js**: Audio visualization and playback
- **Stimulus Controllers**: Event-driven audio control
  - `player_controller.js`: Main player with WaveSurfer integration, queue management, auto-advance
  - `play_pause_controller.js`: Play/pause button states
  - `song_controller.js`: Individual song card interactions
  - `song-list_controller.js`: Queue management
  - `banner_controller.js`: Dynamic banner image/video updates
  - `time_display_controller.js`: Current time and duration display
  - `auto-advance_controller.js`: Automatic progression through playlist
  - `play-on-load_controller.js`: Auto-play behavior management
  - `smart-image_controller.js`: Image loading and fallback handling
  - `sidebar_controller.js`: Navigation state

**Event-driven Communication**: Custom DOM events coordinate between controllers:
- `player:play-requested`: Song selection
- `player:state:changed`: Playback state updates
- `player:queue:updated`: Queue synchronization
- `player:time:update`: Time display updates
- `player:auto-advance:changed`: Auto-advance toggle
- `music:banner:update`: Banner image/video changes
- `audio:changed`, `audio:ended`, `audio:error`: Audio lifecycle events

**S3 Integration**: Large audio files served from S3 with proper CORS and streaming configuration.

**Admin namespace**: All admin functionality under `admin` namespace with Admin model authentication.

**Key models**:
- `Song`: Core entity with audio file, metadata, belongs to Album
- `Album`: Collection of songs, belongs to Artist, has cover image
- `Artist`: Top-level entity with banner image/video
- `Genre`, `SongGenre`: Genre tagging system
- `Playlist`, `PlaylistSong`: User-created playlists

**Routing pattern**: Music browsing at `/music` with sub-routes for artists, genres, playlists

## Design Philosophy

- **Vintage aesthetics**: Polaroid frames, grain effects, old-school video treatment (Salt and Tar)
- **Dark mode support**: Implemented via Stimulus controllers
- **Responsive design**: Mobile-first with Tailwind breakpoints
- **Component architecture**: ViewComponents for reusable UI (Portfolio app)
- **Progressive Web App (PWA)**: Service worker support configured

## Common Development Tasks

### Adding a new sub-project to Portfolio App

1. Create models, controllers, views for the project
2. Add routes (typically as resources or collection routes)
3. Add namespace in `milk_admin` if admin functionality needed
4. Update navigation in shared layouts
5. Configure S3 storage if large files needed

### Adding features to RML Soundscape player

1. Music player changes typically require:
   - Updates to Stimulus controller in `app/javascript/controllers/music/`
   - Custom event coordination between controllers
   - Consider localStorage for persistent user preferences
2. WaveSurfer configuration in `player_controller.js`
3. Queue management logic for auto-advance/shuffle
4. Banner updates via `banner_controller.js`

### Working with S3 Storage

Both apps use Active Storage with S3:
```ruby
# In models
has_one_attached :audio_file
has_one_attached :banner_image
has_one_attached :thumbnail
```

Environment variables required:
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `AWS_REGION`
- `AWS_BUCKET`

### Authentication

Portfolio App:
- Admin: `http://localhost:3000/milk_admins/sign_in`
- User: `http://localhost:3000/users/sign_in`

RML Soundscape:
- Admin: `http://localhost:3000/admins/sign_in`
- Registrations disabled in both apps

## Key Files and Their Purposes

### Portfolio App
- `app/controllers/milk_admin/*`: Admin CRUD operations
- `app/controllers/static_pages_controller.rb`: Main portfolio pages
- `app/controllers/salt_and_tar_controller.rb`: Salt and Tar video pages
- `app/views/static_pages/index.html.erb`: Homepage
- `db/seeds.rb`: Sample video content for Salt and Tar

### RML Soundscape
- `app/javascript/controllers/music/player_controller.js`: Core audio player (550+ lines)
- `app/controllers/music_controller.rb`: Music browsing and player pages
- `app/controllers/admin/songs_controller.rb`: Song upload and management
- `config/storage.yml`: S3 configuration

## Development Notes

- Both apps configured for Heroku deployment (`Procfile`)
- Docker support via Kamal
- Solid Cache, Solid Queue, Solid Cable for Rails 8 modern stack
- Friendly IDs for SEO-friendly URLs (Portfolio app)
- Meta tags and sitemap generation (Portfolio app)
- Ransack for search functionality (Portfolio app)
- PWA manifests configured but currently commented out

## Rails Templates

The founder maintains Rails templates for quick setup:
- Gitignore template: Fixes `vendor/bundle` tracking issue
- Tailwind template: Custom theme setup with markdown guide
- Flash messages template: Pre-configured with Stimulus

## Project Phases

Per the Studio Plan:
- **Phase 1** (Current): Foundation, community building, RML Soundscape MVP
- **Phase 2**: Public launch of Soundscape, begin HermitPlus development
- **Phase 3**: Sustainability, HermitPlus launch, "Sailing Hub" concept

## Testing Approach

Tests are minimal currently. Standard Rails testing:
```bash
bin/rails test
bin/rails test:system
```

Fixtures in `test/fixtures/` for all models.

## Important Conventions

- Use dashes in URLs, not underscores
- Admin authentication required for all admin namespaces
- S3 URLs for production assets
- Local PostgreSQL for development
- Event-driven Stimulus controllers for complex interactions
- Tailwind utility classes, no custom CSS files
- ERB templates, no JSX/React

