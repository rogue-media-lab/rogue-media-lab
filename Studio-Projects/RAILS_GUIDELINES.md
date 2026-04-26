# Rails Development Guidelines

**Purpose:** These guidelines ensure maintainability, safety, and developer velocity across all Rogue Media Lab Rails applications. **MUST** rules are non-negotiable and enforced by CI/tooling. **SHOULD** rules are strong recommendations that improve code quality.

**Tech Stack Reference:**
- Ruby 3.3.7, Rails 8.x (requires Ruby 3.2+)
- PostgreSQL with custom defaults (`postgres`/`postgres` locally)
- Hotwire (Turbo + Stimulus) via Importmap
- Propshaft asset pipeline (NOT Sprockets)
- Tailwind CSS
- Active Storage with S3
- RSpec for testing
- RuboCop for linting
- Authentication: Rails 8 built-in generator OR Devise (legacy)
- Background Jobs: Solid Queue (Rails 8 default) OR ActiveJob with other adapters
- Caching: Solid Cache (Rails 8 default)
- WebSockets: Solid Cable (Rails 8 default)

---

## Rails 8 Specific Features

Rails 8 introduces several new defaults and patterns. This section covers Rails 8-specific conventions.

### R8-1: Use params.expect Instead of require + permit (SHOULD)

Rails 8 introduces `params.expect` which is safer and more concise than the traditional `require` + `permit` pattern.

```ruby
# ✅ Good: Rails 8 params.expect
class PostsController < ApplicationController
  def create
    @post = Post.new(post_params)
    # ...
  end

  private

  def post_params
    params.expect(post: [:title, :body, :published])
  end
end

# ✅ Good: Nested parameters
def post_params
  params.expect(post: [:title, :body, comments: [[:message, :author]]])
end

# ✅ Good: Array of hashes (note double brackets)
def post_params
  params.expect(post: [:title, tags: [[:name, :color]]])
end

# ❌ Old way (still works, but params.expect is preferred)
def post_params
  params.require(:post).permit(:title, :body, :published)
end
```

**Benefits:**
- Better error handling: Returns 400 (Bad Request) instead of 500 (Internal Server Error) for invalid param types
- Structure validation: Enforces expected types (hash vs array)
- More concise syntax

**Use `expect!` for debugging:**
```ruby
# Use expect! to raise exception instead of rendering 400
# Useful for internal APIs where you want stack traces
params.expect!(user: [:name, :email])
```

### R8-2: Built-in Authentication Generator (SHOULD)

Rails 8 includes a built-in authentication generator as an alternative to Devise for simple authentication needs.

```bash
# Generate authentication scaffolding
bin/rails generate authentication
```

**What it generates:**
- `User` model with `has_secure_password`
- `Session` model for database-tracked sessions
- `SessionsController` for login/logout
- `PasswordsController` for password reset
- `Authentication` concern for session management
- Built-in rate limiting (10 requests per 3 minutes per IP)

**When to use:**
- ✅ New Rails 8 applications with simple auth needs
- ✅ You want full control over authentication logic
- ✅ You don't need complex features (OAuth, multi-factor auth, etc.)

**When to use Devise instead:**
- ❌ Existing applications already using Devise
- ❌ Need OAuth/social login
- ❌ Need complex features (confirmable, lockable, etc.)

```ruby
# Example: Using Rails 8 authentication in controllers
class ApplicationController < ActionController::Base
  include Authentication

  # Authentication concern provides:
  # - authenticate (before_action to require login)
  # - current_user
  # - user_signed_in?
end

class PostsController < ApplicationController
  before_action :authenticate

  def index
    @posts = current_user.posts
  end
end
```

### R8-3: Solid Queue for Background Jobs (Rails 8 Default)

Solid Queue is the Rails 8 default for background jobs, replacing the need for Redis/Sidekiq.

```ruby
# ✅ Good: Standard ActiveJob (works with Solid Queue)
class AudioAnalysisJob < ApplicationJob
  queue_as :default

  def perform(song)
    # Heavy processing...
    metadata = extract_metadata(song.audio_file)
    song.update(duration_seconds: metadata[:duration])
  end
end

# Usage (same as before)
AudioAnalysisJob.perform_later(song)
```

**Configuration:**
```yaml
# config/solid_queue.yml (auto-generated in Rails 8)
production:
  dispatchers:
    - polling_interval: 1
      batch_size: 500
  workers:
    - queues: "*"
      threads: 3
      processes: 1
      polling_interval: 0.1
```

**Database setup:**
Solid Queue uses a separate database by default (see `config/database.yml` queue section).

**Running in production:**
```bash
# Via Puma plugin (default)
bundle exec puma

# OR dedicated process
bin/jobs
```

### R8-4: Solid Cache for Caching (Rails 8 Default)

Solid Cache replaces Redis/Memcached with database-backed caching.

```ruby
# config/environments/production.rb
config.cache_store = :solid_cache_store

# Usage (standard Rails caching)
Rails.cache.fetch("user_#{user.id}_posts", expires_in: 1.hour) do
  user.posts.to_a
end

# Fragment caching in views (works the same)
<% cache @post do %>
  <%= render @post %>
<% end %>
```

**Benefits:**
- No Redis/Memcached dependency
- Uses SSDs for fast disk-based caching
- Larger cache sizes possible
- Works with existing Rails caching APIs

### R8-5: Solid Cable for WebSockets (Rails 8 Default)
Solid Cable replaces Redis for ActionCable WebSocket communication.

```ruby
# config/cable.yml (Rails 8 default)
production:
  adapter: solid_cable
  polling_interval: 0.1
  message_retention: 1.day

# Usage (standard ActionCable)
class NotificationChannel < ApplicationCable::Channel
  def subscribed
    stream_from "notifications:#{current_user.id}"
  end
end

# Broadcasting (works the same)
ActionCable.server.broadcast(
  "notifications:#{user.id}",
  { message: "New notification!" }
)
```

**Database setup:**
Solid Cable uses a separate database by default (see `config/database.yml` cable section).

### R8-6: Propshaft Asset Pipeline & the `:app` Symbol (MUST)
Rails 8 uses Propshaft as the default asset pipeline (replacing Sprockets). Propshaft has a key convention for loading stylesheets.

**The `:app` symbol for stylesheet_link_tag:**
```erb
# ✅ Correct: Loads ALL CSS files from app/assets/stylesheets/ automatically
<%= stylesheet_link_tag :app, "data-turbo-track": "reload" %>

# ❌ Wrong: Only loads a single named file (misses actiontext.css, etc.)
<%= stylesheet_link_tag "application", "data-turbo-track": "reload" %>
```

**Why this matters:**
- `:app` tells Propshaft to bundle and load ALL `.css` files in `app/assets/stylesheets/`
- This includes `application.css`, `actiontext.css`, and any other stylesheets
- Using a string like `"application"` only loads that ONE file
- Action Text icons, Turbo native styles, and other gem assets rely on this convention

**When `:app` is needed:**
- Admin layouts that use Action Text (Trix editor)
- Any layout that needs stylesheets from gems (Action Text, Turbo Native, etc.)
- Default Rails 8 layouts use `:app` - follow this pattern

**Propshaft vs Sprockets:**
- Propshaft: No `require` statements in CSS files. Just drop files in `app/assets/stylesheets/`
- Sprockets (old way): Required `@import` and `require` directives in manifest files
- Propshaft is simpler but you MUST use `:app` to load the full stylesheet bundle

### R8-7: Importmap Configuration for Stimulus & JavaScript (SHOULD)
Rails 8 uses importmap for managing JavaScript dependencies without Node.js/npm.

**Setup pattern:**
```bash
# Pin a new dependency
bin/importmap pin @hotwired/turbo-rails
bin/importmap pin @hotwired/stimulus

# Verify pins
bin/importmap json
```

**Required file structure:**
```
config/importmap.rb              # Pin definitions
app/javascript/
  application.js                 # Main entry point
  controllers/
    application.js               # Stimulus application setup
    index.js                     # Controller registration
```

**`app/javascript/controllers/application.js`:**
```javascript
import { Application } from "@hotwired/stimulus"
const application = Application.start()
export { application }
```

**`app/javascript/controllers/index.js`:**
```javascript
import { application } from "controllers/application"
// Eager load all controllers
import "./**/*_controller"
```

**`app/javascript/application.js`:**
```javascript
import "@hotwired/turbo-rails"
import "controllers"
```

**Key convention:**
- Controllers are eager-loaded via `import "./**/*_controller"` - any file matching `*_controller.js` in the controllers directory is automatically registered
- No need for manual registration of each controller
- Controller names map to data attributes: `app/javascript/controllers/player_controller.js` → `data-controller="player"`

**Verifying importmap is working:**
```erb
# In your layout:
<%= javascript_importmap_tags %>
```

**Common mistake:** Forgetting to pin a module before importing it. Always `bin/importmap pin` first.

### R8-8: Use Single Database for Solid Trifecta (RECOMMENDED)

**Rails 8 ships with separate databases by default, but this creates deployment hassles on Heroku, Render, and DigitalOcean. For most applications, consolidating into a single database is simpler and sufficient.**

**Why single database?**
- ✅ Simpler deployment (one database to manage)
- ✅ Lower infrastructure costs (no need for multiple database instances)
- ✅ Easier backups and migrations
- ✅ Sufficient for small to medium traffic applications
- ✅ PostgreSQL handles this well (unlike SQLite)

**When to use separate databases:**
- High-traffic applications with heavy job/cache/websocket load
- SQLite in production (write contention issues)
- Need strict separation for compliance/security

#### Step-by-Step Configuration

**Step 1: Generate migrations for Solid gems**
```bash
bin/rails g migration solid_cable
bin/rails g migration solid_queue
bin/rails g migration solid_cache
```

**Step 2: Copy schema into migrations**
Copy the table definitions from:
- `db/cable_schema.rb` → into `solid_cable` migration
- `db/queue_schema.rb` → into `solid_queue` migration
- `db/cache_schema.rb` → into `solid_cache` migration

**Step 3: Delete schema files**
```bash
rm db/cable_schema.rb
rm db/queue_schema.rb
rm db/cache_schema.rb
```

**Step 4: Run migrations**
```bash
bin/rails db:migrate
```

**Step 5: Update database.yml**
Remove `cache`, `queue`, and `cable` database entries:

```yaml
# config/database.yml
production:
  primary:
    <<: *default
    database: myapp_production
    username: myapp
    password: <%= ENV["DATABASE_PASSWORD"] %>

# Remove these sections:
# queue:
# cache:
# cable:
```

**Step 6: Update cable.yml**
```yaml
# config/cable.yml
production:
  adapter: solid_cable
  connects_to:
    database:
      writing: :primary  # Changed from :cable
  polling_interval: 0.1
  message_retention: 1.day
```

**Step 7: Update cache.yml**
```yaml
# config/cache.yml
production:
  primary:
    database: :primary  # Changed from :cache
    store_options:
      max_age: <%= 60.days.to_i %>
      max_size: <%= 256.megabytes %>
```

**Step 8: Update production.rb**
```ruby
# config/environments/production.rb

# Remove this line:
# config.solid_queue.connects_to = { database: { writing: :queue } }

# Keep these:
config.cache_store = :solid_cache_store
config.active_job.queue_adapter = :solid_queue
```

**Step 9: Update development.rb (optional)**
```ruby
# config/environments/development.rb

# Change from :inline to :solid_queue to match production behavior
config.active_job.queue_adapter = :solid_queue
```

#### Verification

After deploying, verify everything works:

```ruby
# Test caching
Rails.cache.write("test", "value")
Rails.cache.read("test") # Should return "value"

# Test background jobs
class TestJob < ApplicationJob
  def perform
    Rails.logger.info "Job ran successfully!"
  end
end
TestJob.perform_later

# Test ActionCable (if using)
# Send a test broadcast and check it's received
```

#### Database Inspection

All tables should now be in your primary database:

```bash
# PostgreSQL
psql myapp_production -c "\dt solid_*"

# Should show:
# solid_cable_messages
# solid_queue_*
# solid_cache_entries
```

**For complete walkthrough, see:** https://masonroberts.substack.com/p/run-the-solid-trifecta-in-a-single

### R8-9: has_many :through Associations — The Missing Source Gotcha (SHOULD)

**When defining `has_many :through` associations, Rails must be able to trace the path through the join model. If the association name on the join model doesn't match the target model's name, you MUST specify `source:`.**

```ruby
# The problem: join model uses `volunteer_submission` but we want `submissions`
class VolunteerSubmissionInterest < ApplicationRecord
  belongs_to :volunteer_submission  # ← this is the actual association name
  belongs_to :volunteer_interest
end

# ❌ BROKEN: Rails looks for `submission` on join model (doesn't exist)
class VolunteerInterest < ApplicationRecord
  has_many :volunteer_submission_interests
  has_many :submissions, through: :volunteer_submission_interests
  # Error: "Could not find the source association(s) 'submission' or :submissions"
end

# ✅ FIXED: Tell Rails which association on the join model to use
class VolunteerInterest < ApplicationRecord
  has_many :volunteer_submission_interests
  has_many :submissions, through: :volunteer_submission_interests, source: :volunteer_submission
end
```

**When this error occurs:**
- `ActiveRecord::HasManyThroughSourceAssociationNotFoundError`
- Error message: "Could not find the source association(s) 'X' or :X in model Y"

**How to debug:**
1. Read the error message — it names the join model and the missing association
2. Check the join model's `belongs_to` declarations
3. If the `belongs_to` name differs from what `has_many :through` expects, add `source:`
4. Test with a quick `bin/rails runner "Model.association_name.count"` before using in views

**Naming convention that avoids this:**
- If join model's `belongs_to :widget` matches the target model, no `source:` needed
- If join model's `belongs_to :special_widget` but target is `Widget`, use `source: :special_widget`

---

## 1. Before Writing Code

### BP-1: Clarify Requirements (MUST)
Ask clarifying questions before coding. Never make assumptions about:
- User authentication flows
- Data relationships (has_many, belongs_to, polymorphic)
- UI/UX expectations (Turbo Frames? Full page loads?)
- Storage requirements (local vs S3)

### BP-2: Draft an Approach (SHOULD)
For complex features, outline your approach:
- **Models**: What data structures? Associations? Validations?
- **Controllers**: RESTful? Nested resources? Admin namespace?
- **Views**: Turbo Frames? Stimulus controllers? Partials?
- **Services**: Does this need a service object or is it model logic?

```ruby
# Example outline comment before coding:
# Approach:
# - Add `has_many :tracks` to Album model
# - Create TracksController nested under albums
# - Use Turbo Frame for inline track editing
# - Add track_controller.js for drag-and-drop reordering
```

### BP-3: Compare Alternatives (SHOULD)
If ≥2 approaches exist, list pros/cons:

**Example: Where to put playlist generation logic?**
- **Option A: Model method** `Playlist#generate_smart_playlist`
  - ✅ Easily testable, keeps business logic with the data
  - ❌ Could bloat model if complex
- **Option B: Service object** `SmartPlaylistGenerator`
  - ✅ Isolated, single responsibility
  - ❌ Adds another class to maintain

---

## 2. While Coding

### C-1: Test-Driven Development (MUST)
Follow Red-Green-Refactor:
1. Write failing test
2. Implement minimum code to pass
3. Refactor if needed

```ruby
# ✅ Good: Test first
describe Song, type: :model do
  it "generates a permalink from title" do
    song = create(:song, title: "Hello World")
    expect(song.permalink).to eq("hello-world")
  end
end

# Then implement:
class Song < ApplicationRecord
  before_save :generate_permalink

  private

  def generate_permalink
    self.permalink = title.parameterize
  end
end
```

### C-2: Use Domain Vocabulary (MUST)
Name things using existing domain language for consistency:
- Follow Rails conventions: `snake_case` for methods/variables, `CamelCase` for classes/modules
- Use domain terms: `Playlist`, `Track`, `Album`, not `MusicCollection`, `AudioFile`, `MusicGroup`
- Match your database schema names

```ruby
# ✅ Good: Domain language
class TrainingSession < ApplicationRecord
  belongs_to :scenario
  has_many :messages
end

# ❌ Bad: Generic/unclear
class Session < ApplicationRecord
  belongs_to :thing
  has_many :items
end
```

### C-3: Don't Over-Extract (SHOULD NOT)
Avoid creating Service Objects, Form Objects, or POROs when simple model/controller methods suffice:

```ruby
# ✅ Good: Simple model method
class Album < ApplicationRecord
  def total_duration
    songs.sum(:duration_seconds)
  end
end

# ❌ Bad: Unnecessary service object
class AlbumDurationCalculator
  def initialize(album)
    @album = album
  end

  def calculate
    @album.songs.sum(:duration_seconds)
  end
end
```

**When to extract a Service Object:**
- Logic spans multiple models/resources
- Complex external API integration
- Multi-step business process (e.g., `OrderFulfillmentService`)
- Heavy algorithmic work that doesn't fit model responsibility

### C-4: Prefer Simple, Composable Methods (SHOULD)
Write small, testable methods that do one thing well:

```ruby
# ✅ Good: Composable methods
class PlaylistGenerator
  def generate(user, mood:)
    songs = songs_matching_mood(mood)
    songs = filter_by_user_preferences(songs, user)
    songs.sample(20)
  end

  private

  def songs_matching_mood(mood)
    Song.joins(:genres).where(genres: { mood: mood })
  end

  def filter_by_user_preferences(songs, user)
    songs.where.not(artist_id: user.blocked_artist_ids)
  end
end

# ❌ Bad: Monolithic method
class PlaylistGenerator
  def generate(user, mood:)
    # 50 lines of nested conditionals and queries
  end
end
```

### C-5: Use Value Objects for Complex Data (MUST)
Use Ruby Structs or Value Objects instead of passing hashes around:

```ruby
# ✅ Good: Struct for type-safe data passing
AudioMetadata = Struct.new(:duration, :bitrate, :format, keyword_init: true)

class AudioProcessor
  def process(metadata)
    return unless metadata.format == 'mp3'
    # ...
  end
end

# ❌ Bad: Hash with unclear structure
class AudioProcessor
  def process(data)
    return unless data[:format] == 'mp3'  # Typo-prone, no type safety
    # ...
  end
end
```

### C-6: Trust Rails Autoloading (MUST)
Don't manually `require` files from `app/**` - Rails autoloads them:

```ruby
# ✅ Good: Rails autoloads app/services/ai_conversation_service.rb
class SomeController < ApplicationController
  def create
    AiConversationService.new.call
  end
end

# ❌ Bad: Unnecessary require
require_relative '../services/ai_conversation_service'
```

**Exception:** Explicitly require gems or lib files not in Rails autoload paths.

### C-7: Comments Are Code Smell (SHOULD NOT)
Write self-documenting code. Comments are only acceptable for:
- Explaining non-obvious business rules
- Complex regex patterns
- Deliberate deviation from Rails conventions
- TODO/FIXME with ticket references

```ruby
# ✅ Good: Self-documenting
def active_subscribers
  users.where(subscription_status: 'active')
end

# ❌ Bad: Comment explains what code should show
def get_users
  # Get all users who are subscribed
  users.where(subscription_status: 'active')
end

# ✅ Acceptable: Business rule explanation
def apply_early_bird_discount(price)
  # Per 2024 Q4 promotion: 15% off for purchases before 9am EST
  Time.current.hour < 9 ? price * 0.85 : price
end
```

### C-8: Prefer Rails/Ruby Built-ins (SHOULD)
Use ActiveRecord, Enumerable, and Ruby stdlib before adding gems:

```ruby
# ✅ Good: ActiveRecord scopes
class Song < ApplicationRecord
  scope :by_artist, ->(artist) { where(artist: artist) }
  scope :recent, -> { where('created_at > ?', 1.week.ago) }
end

# ❌ Bad: External gem for simple queries
gem 'query_builder_extreme'
```

**Common Rails features to leverage:**
- ActiveRecord scopes, enums, callbacks
- ActiveSupport core extensions (`1.week.ago`, `.presence`, `.try`)
- ActionView helpers (`link_to`, `form_with`)
- ActiveJob for background tasks
- ActionMailer for emails

### C-9: Extract Only When Necessary (SHOULD NOT)
Don't extract methods/classes unless:
1. Logic is reused in ≥2 places
2. Extraction enables unit testing otherwise untestable code
3. Original code is genuinely unreadable without extraction

```ruby
# ✅ Good: Don't extract if used once and simple
def total_price
  base_price + (base_price * tax_rate)
end

# ❌ Bad: Unnecessary extraction
def total_price
  calculate_total
end

def calculate_total
  base_price + calculate_tax
end

def calculate_tax
  base_price * tax_rate
end
```

---

## 3. Testing with RSpec

### T-1: Test Location (MUST)
Follow RSpec Rails conventions:
- **Models**: `spec/models/song_spec.rb`
- **Controllers**: `spec/controllers/music_controller_spec.rb` OR `spec/requests/music_spec.rb` (prefer requests)
- **Services**: `spec/services/ai_conversation_service_spec.rb`
- **Lib/POROs**: `spec/lib/playlist_generator_spec.rb`
- **Features**: `spec/features/user_plays_song_spec.rb`
- **System**: `spec/system/admin_uploads_song_spec.rb`

### T-2: Test the Right Layer (MUST)
Choose the right test type:

**Model Specs** - Business logic, validations, associations
```ruby
# spec/models/album_spec.rb
RSpec.describe Album, type: :model do
  it { should have_many(:songs) }
  it { should validate_presence_of(:title) }

  describe '#total_duration' do
    it 'sums all song durations' do
      album = create(:album)
      create(:song, album: album, duration_seconds: 180)
      create(:song, album: album, duration_seconds: 200)

      expect(album.total_duration).to eq(380)
    end
  end
end
```

**Request Specs** - API/Controller responses (prefer over controller specs)
```ruby
# spec/requests/api/songs_spec.rb
RSpec.describe 'Songs API', type: :request do
  describe 'GET /api/songs' do
    it 'returns all songs as JSON' do
      create_list(:song, 3)

      get api_songs_path, headers: { 'Accept' => 'application/json' }

      expect(response).to have_http_status(:ok)
      expect(JSON.parse(response.body).size).to eq(3)
    end
  end
end
```

**System/Feature Specs** - Full user workflows with JavaScript
```ruby
# spec/system/user_plays_song_spec.rb
RSpec.describe 'Playing songs', type: :system, js: true do
  it 'plays a song when user clicks play button' do
    song = create(:song, title: 'Test Song')

    visit music_path
    click_button 'Play'

    expect(page).to have_css('.player.playing')
    expect(page).to have_content('Test Song')
  end
end
```

### T-3: Separate Unit from Integration (MUST)
- **Unit tests**: Fast, no database, test logic in isolation
- **Integration tests**: Touch database, test component interactions

```ruby
# ✅ Good: Unit test (no DB)
RSpec.describe PlaylistSorter do
  describe '#sort_by_mood' do
    it 'sorts songs by mood score' do
      songs = [
        Song.new(mood_score: 3),
        Song.new(mood_score: 1),
        Song.new(mood_score: 2)
      ]

      result = described_class.new.sort_by_mood(songs)

      expect(result.map(&:mood_score)).to eq([1, 2, 3])
    end
  end
end

# ✅ Good: Integration test (with DB)
RSpec.describe Song, type: :model do
  describe '#generate_waveform' do
    it 'creates waveform data from audio file', :integration do
      song = create(:song, :with_audio_file)

      song.generate_waveform

      expect(song.waveform_data).to be_present
    end
  end
end
```

### T-4: Prefer Integration Over Heavy Mocking (SHOULD)
Don't mock ActiveRecord or Rails core - use request/system specs instead:

```ruby
# ✅ Good: Integration test
RSpec.describe 'Admin uploads song', type: :request do
  it 'creates song with audio file' do
    sign_in create(:admin)

    post admin_songs_path, params: {
      song: {
        title: 'New Song',
        audio_file: fixture_file_upload('song.mp3', 'audio/mpeg')
      }
    }

    expect(Song.last.title).to eq('New Song')
    expect(Song.last.audio_file).to be_attached
  end
end

# ❌ Bad: Heavy mocking
RSpec.describe SongsController do
  it 'creates song' do
    allow(Song).to receive(:create).and_return(double(id: 1))
    # This doesn't test anything meaningful
  end
end
```

**When mocking IS appropriate:**
- External APIs (don't hit real APIs in tests)
- Time-based logic (`travel_to`)
- File system operations
- Third-party services (S3, payment gateways)

### T-5: Parameterized Tests for Algorithms (SHOULD)
Test edge cases with table-driven tests:

```ruby
# ✅ Good: Test multiple cases
RSpec.describe PriceCalculator do
  describe '#calculate_discount' do
    test_cases = [
      { price: 100, tier: :basic, expected: 90 },
      { price: 100, tier: :premium, expected: 80 },
      { price: 50, tier: :basic, expected: 47.5 },
      { price: 0, tier: :premium, expected: 0 }
    ]

    test_cases.each do |test_case|
      it "returns #{test_case[:expected]} for $#{test_case[:price]} #{test_case[:tier]}" do
        result = described_class.calculate_discount(
          test_case[:price],
          test_case[:tier]
        )
        expect(result).to eq(test_case[:expected])
      end
    end
  end
end
```

### T-6: Single Comprehensive Assertion (SHOULD)
Test entire structures in one assertion when possible:

```ruby
# ✅ Good: Single assertion
expect(response_json).to match(
  'id' => song.id,
  'title' => 'Test Song',
  'duration' => 180,
  'artist' => { 'name' => 'Test Artist' }
)

# ❌ Bad: Multiple fragile assertions
expect(response_json['id']).to eq(song.id)
expect(response_json['title']).to eq('Test Song')
expect(response_json['duration']).to eq(180)
expect(response_json.dig('artist', 'name')).to eq('Test Artist')
```

---

## 4. Database & ActiveRecord

### D-1: Use Transactions for Atomicity (MUST)
Wrap multi-step database operations in transactions:

```ruby
# ✅ Good: Atomic operation
ActiveRecord::Base.transaction do
  album = Album.create!(title: 'New Album')
  song.update!(album: album)
  PlaylistSong.where(song: song).destroy_all
end

# ❌ Bad: Non-atomic (album created even if song update fails)
album = Album.create!(title: 'New Album')
song.update!(album: album)
PlaylistSong.where(song: song).destroy_all
```

### D-2: Strong Parameters (MUST)
Always use strong parameters in controllers:

```ruby
# ✅ Best: Rails 8 params.expect (preferred)
class SongsController < ApplicationController
  def create
    @song = Song.new(song_params)
    # ...
  end

  private

  def song_params
    params.expect(song: [:title, :duration_seconds, :album_id])
  end
end

# ✅ Good: Traditional require + permit (still works)
def song_params
  params.require(:song).permit(:title, :duration_seconds, :album_id)
end

# ❌ Bad: Mass assignment vulnerability
def create
  @song = Song.new(params[:song])
end
```

**See R8-1 for more details on `params.expect`.**

### D-3: Use Scopes for Reusable Queries (SHOULD)
Define named scopes for common queries:

```ruby
# ✅ Good: Reusable, chainable scopes
class Song < ApplicationRecord
  scope :published, -> { where(published: true) }
  scope :by_genre, ->(genre) { joins(:genres).where(genres: { name: genre }) }
  scope :recent, -> { where('created_at > ?', 1.month.ago) }
end

# Usage:
Song.published.by_genre('rock').recent

# ❌ Bad: Query logic scattered in controllers
# app/controllers/songs_controller.rb
@songs = Song.where(published: true).joins(:genres).where(genres: { name: 'rock' })
```

### D-4: N+1 Query Prevention (MUST)
Use `includes`, `preload`, or `eager_load` to avoid N+1 queries:

```ruby
# ✅ Good: Eager loading
@songs = Song.includes(:artist, :album).all

# View:
@songs.each do |song|
  song.artist.name  # No additional query
  song.album.title  # No additional query
end

# ❌ Bad: N+1 queries
@songs = Song.all

# View triggers N queries:
@songs.each do |song|
  song.artist.name  # Query!
  song.album.title  # Query!
end
```

**Use Bullet gem in development to detect N+1s.**

### D-5: Database Indexes (MUST)
Add indexes for foreign keys and frequently queried columns:

```ruby
# ✅ Good: Migration with indexes
class CreateSongs < ActiveRecord::Migration[8.0]
  def change
    create_table :songs do |t|
      t.string :title, null: false
      t.references :album, null: false, foreign_key: true  # Automatic index
      t.references :artist, null: false, foreign_key: true
      t.integer :duration_seconds
      t.boolean :published, default: false

      t.timestamps
    end

    # Add index for common query
    add_index :songs, :published
    add_index :songs, [:artist_id, :published]  # Compound index
  end
end
```

### D-6: Model Validations (SHOULD)
Validate at the model level for data integrity:

```ruby
# ✅ Good: Model validations
class Song < ApplicationRecord
  belongs_to :album
  belongs_to :artist

  validates :title, presence: true, length: { maximum: 200 }
  validates :duration_seconds, numericality: { greater_than: 0, allow_nil: true }
  validates :permalink, uniqueness: { scope: :artist_id }

  validate :audio_file_format

  private

  def audio_file_format
    return unless audio_file.attached?

    unless audio_file.content_type.in?(%w[audio/mpeg audio/mp4])
      errors.add(:audio_file, 'must be MP3 or M4A')
    end
  end
end
```

---

## 5. Hotwire (Turbo + Stimulus)

### H-1: Use Turbo Frames for Partial Updates (SHOULD)
Avoid full page reloads for interactive components:

```erb
<%# ✅ Good: Turbo Frame for inline editing %>
<%= turbo_frame_tag "song_#{song.id}" do %>
  <div class="song-card">
    <h3><%= song.title %></h3>
    <%= link_to "Edit", edit_admin_song_path(song) %>
  </div>
<% end %>

<%# edit.html.erb wraps form in matching turbo_frame_tag %>
<%= turbo_frame_tag "song_#{@song.id}" do %>
  <%= form_with model: [:admin, @song] do |f| %>
    <%= f.text_field :title %>
    <%= f.submit "Save" %>
  <% end %>
<% end %>
```

### H-2: Stimulus Controller Organization (MUST)
Organize Stimulus controllers by domain/feature:

```
app/javascript/controllers/
  music/
    player_controller.js       # Main audio player
    playlist_controller.js     # Playlist interactions
    waveform_controller.js     # WaveSurfer integration
  admin/
    form_controller.js         # Admin form behaviors
    upload_controller.js       # File upload progress
  shared/
    modal_controller.js        # Reusable modal
    dropdown_controller.js     # Reusable dropdown
```

### H-3: Stimulus Actions & Targets (SHOULD)
Use clear action and target naming:

```javascript
// ✅ Good: Clear naming
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = ["playButton", "pauseButton", "progressBar"]
  static values = { songId: String }

  connect() {
    // Setup
  }

  play(event) {
    event.preventDefault()
    this.playButtonTarget.classList.add("hidden")
    this.pauseButtonTarget.classList.remove("hidden")
    // Play audio...
  }

  pause(event) {
    event.preventDefault()
    this.pauseButtonTarget.classList.add("hidden")
    this.playButtonTarget.classList.remove("hidden")
    // Pause audio...
  }

  updateProgress(event) {
    const percent = (event.detail.currentTime / event.detail.duration) * 100
    this.progressBarTarget.style.width = `${percent}%`
  }
}
```

```erb
<div data-controller="player" data-player-song-id-value="<%= song.id %>">
  <button
    data-player-target="playButton"
    data-action="click->player#play">
    Play
  </button>
  <button
    data-player-target="pauseButton"
    data-action="click->player#pause"
    class="hidden">
    Pause
  </button>
  <div data-player-target="progressBar"></div>
</div>
```

### H-4: Custom Events for Controller Communication (SHOULD)
Use custom DOM events for loosely coupled Stimulus controllers:

```javascript
// ✅ Good: Event-driven communication
// player_controller.js
play(event) {
  this.audio.play()

  // Dispatch custom event
  this.dispatch("playing", {
    detail: { songId: this.songIdValue }
  })
}

// playlist_controller.js
connect() {
  this.element.addEventListener("player:playing", this.updateNowPlaying.bind(this))
}

updateNowPlaying(event) {
  const songId = event.detail.songId
  // Update UI...
}
```

### H-5: Turbo Streams for Live Updates (SHOULD)
Use Turbo Streams for real-time updates from server:

```ruby
# ✅ Good: Controller responds with Turbo Stream
class Admin::SongsController < ApplicationController
  def create
    @song = Song.new(song_params)

    if @song.save
      respond_to do |format|
        format.turbo_stream do
          render turbo_stream: turbo_stream.append(
            "songs_list",
            partial: "admin/songs/song",
            locals: { song: @song }
          )
        end
        format.html { redirect_to admin_songs_path }
      end
    end
  end
end
```

---

## 6. Active Storage & S3

### S-1: Use Variants for Image Sizes (SHOULD)
Define image variants instead of creating multiple attachments:

```ruby
# ✅ Good: Variants
class Album < ApplicationRecord
  has_one_attached :cover_image

  def cover_thumb
    cover_image.variant(resize_to_limit: [200, 200])
  end

  def cover_large
    cover_image.variant(resize_to_limit: [800, 800])
  end
end

# View:
<%= image_tag album.cover_thumb %>
<%= image_tag album.cover_large %>
```

### S-2: Validate Attachments (MUST)
Validate file types and sizes:

```ruby
# ✅ Good: Attachment validations (requires gem 'active_storage_validations')
class Song < ApplicationRecord
  has_one_attached :audio_file

  validates :audio_file,
    content_type: ['audio/mpeg', 'audio/mp4', 'audio/flac'],
    size: { less_than: 100.megabytes }
end

# Alternative: Custom validation without gem (see D-6 for example)
```

### S-3: Direct Uploads for Large Files (SHOULD)
Use Direct Upload for large files (audio, video):

```erb
<%# ✅ Good: Direct upload to S3 %>
<%= form_with model: @song do |f| %>
  <%= f.file_field :audio_file, direct_upload: true %>
<% end %>
```

### S-4: Background Processing for Heavy Operations (MUST)
Process large files in background jobs:

```ruby
# ✅ Good: Background job
class Song < ApplicationRecord
  has_one_attached :audio_file

  after_commit :analyze_audio, on: [:create, :update], if: -> { audio_file.attached? }

  private

  def analyze_audio
    AudioAnalysisJob.perform_later(self)
  end
end

# app/jobs/audio_analysis_job.rb
class AudioAnalysisJob < ApplicationJob
  queue_as :default

  def perform(song)
    # Heavy processing...
    metadata = extract_metadata(song.audio_file)
    song.update(duration_seconds: metadata[:duration])
  end
end
```

---

## 7. Authentication & Authorization

### A-1: Devise Conventions (MUST)
Follow Devise patterns for authentication:

```ruby
# ✅ Good: Devise in controller
class Admin::SongsController < ApplicationController
  before_action :authenticate_admin!

  def index
    @songs = Song.all
  end
end

# ✅ Good: Conditional view logic
<% if admin_signed_in? %>
  <%= link_to "Edit", edit_admin_song_path(song) %>
<% end %>
```

### A-2: Authorization Checks (MUST)
Verify authorization, not just authentication:

```ruby
# ✅ Good: Authorization check
class PlaylistsController < ApplicationController
  before_action :authenticate_user!
  before_action :set_playlist

  def edit
    unless @playlist.user == current_user
      redirect_to playlists_path, alert: "Not authorized"
    end
  end

  private

  def set_playlist
    @playlist = Playlist.find(params[:id])
  end
end

# Better: Use Pundit or CanCanCan for complex authorization
```

### A-3: Namespace Admin Controllers (SHOULD)
Keep admin functionality in separate namespace:

```ruby
# ✅ Good: Admin namespace
# app/controllers/admin/songs_controller.rb
class Admin::SongsController < ApplicationController
  before_action :authenticate_admin!
  layout 'admin'

  def index
    @songs = Song.all
  end
end

# config/routes.rb
namespace :admin do
  resources :songs
  resources :albums
end
```

---

## 8. Code Organization

### O-1: Follow Rails Conventions (MUST)
Respect the Rails directory structure:

- `app/models` - ActiveRecord models, business logic
- `app/controllers` - Request handling, response rendering
- `app/views` - Templates (ERB, partials)
- `app/services` - Complex business logic spanning multiple models
- `app/jobs` - Background jobs (ActiveJob)
- `app/mailers` - Email logic (ActionMailer)
- `app/helpers` - View helpers
- `lib` - Non-Rails code, utilities (not autoloaded by default)

### O-2: Service Objects (SHOULD)
Use service objects for complex, multi-step operations:

```ruby
# ✅ Good: Service object for complex operation
# app/services/ai_conversation_service.rb
class AiConversationService
  def initialize(session, user_message)
    @session = session
    @user_message = user_message
  end

  def call
    create_user_message
    ai_response = fetch_ai_response
    create_ai_message(ai_response)
    update_session_stats

    ai_response
  end

  private

  def create_user_message
    @session.messages.create!(
      role: 'user',
      content: @user_message,
      sequence_number: next_sequence
    )
  end

  # ... other private methods
end

# Controller:
class ConversationsController < ApplicationController
  def create
    response = AiConversationService.new(
      current_session,
      params[:message]
    ).call

    render json: { response: response }
  end
end
```

### O-3: Concerns for Shared Behavior (SHOULD)
Use concerns for shared model/controller behavior:

```ruby
# ✅ Good: Concern for shared behavior
# app/models/concerns/publishable.rb
module Publishable
  extend ActiveSupport::Concern

  included do
    scope :published, -> { where(published: true) }
    scope :draft, -> { where(published: false) }
  end

  def publish!
    update!(published: true, published_at: Time.current)
  end

  def draft?
    !published
  end
end

# Models:
class Song < ApplicationRecord
  include Publishable
end

class Album < ApplicationRecord
  include Publishable
end
```

### O-4: Query Objects for Complex Queries (SHOULD)
Extract complex queries into query objects:

```ruby
# ✅ Good: Query object
# app/queries/smart_playlist_query.rb
class SmartPlaylistQuery
  def initialize(user, mood:, genre: nil, min_rating: nil)
    @user = user
    @mood = mood
    @genre = genre
    @min_rating = min_rating
  end

  def call
    songs = Song.published
    songs = songs.by_mood(@mood) if @mood.present?
    songs = songs.by_genre(@genre) if @genre.present?
    songs = songs.where('rating >= ?', @min_rating) if @min_rating.present?
    songs = songs.where.not(artist_id: @user.blocked_artist_ids)
    songs.order('RANDOM()').limit(20)
  end
end

# Usage:
songs = SmartPlaylistQuery.new(current_user, mood: 'energetic', min_rating: 4).call
```

---

## 9. Tooling & CI Gates

### G-1: RuboCop (MUST)
All code must pass RuboCop:

```bash
# Run locally before committing
bundle exec rubocop

# Auto-fix safe violations
bundle exec rubocop -a
```

**Key RuboCop rules to follow:**
- 2 space indentation
- No trailing whitespace
- 120 character line length (configurable)
- Prefer `do..end` for multi-line blocks, `{..}` for single-line

### G-2: RSpec Test Suite (MUST)
All tests must pass:

```bash
# Run full suite
bundle exec rspec

# Run specific file
bundle exec rspec spec/models/song_spec.rb

# Run specific test
bundle exec rspec spec/models/song_spec.rb:15
```

### G-3: Database Schema Check (MUST)
Ensure `schema.rb` is up to date:

```bash
# After creating migrations
bin/rails db:migrate
bin/rails db:schema:dump
git add db/schema.rb
```

**Never manually edit `schema.rb` - always use migrations.**

### G-4: Brakeman Security Scan (SHOULD)
Run security checks before deploying:

```bash
bundle exec brakeman
```

---

## 10. Git & Commits

### GH-1: Conventional Commits (MUST)
Use Conventional Commits format: https://www.conventionalcommits.org/

```bash
# ✅ Good commit messages:
feat: add playlist shuffle functionality
fix: resolve audio file upload timeout on S3
refactor: extract playlist generation into service object
test: add request specs for admin song upload
docs: update README with S3 configuration steps
chore: upgrade Rails to 8.0.2

# ❌ Bad commit messages:
update stuff
fix bug
wip
changes
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `refactor`: Code restructure without changing behavior
- `test`: Adding or updating tests
- `docs`: Documentation changes
- `chore`: Dependency updates, config changes
- `perf`: Performance improvements
- `style`: Code style changes (formatting, missing semicolons)

### GH-2: Don't Reference AI Tools (SHOULD NOT)
Don't mention Claude/AI in commit messages:

```bash
# ✅ Good
feat: add audio waveform visualization

# ❌ Bad
feat: Claude added audio waveform visualization
fix: AI suggested fix for N+1 query
```

### GH-3: Atomic Commits (SHOULD)
One logical change per commit:

```bash
# ✅ Good: Separate commits
git commit -m "feat: add Song model with validations"
git commit -m "test: add model specs for Song"
git commit -m "feat: add SongsController with CRUD actions"

# ❌ Bad: Everything in one commit
git commit -m "add songs feature with tests and controller"
```

---

## Writing Quality Functions/Methods Checklist

When evaluating a method you've written, ask:

1. **Is it readable?** Can you follow what it does without a comment? If yes, stop here.
2. **Cyclomatic complexity?** Count `if/else`, `case`, `loops` - high complexity = refactor candidate.
3. **Can Ruby/Rails idioms simplify it?** Consider `&:method`, `map`, `select`, `find_by`, etc.
4. **Unused parameters?** Remove them or use them.
5. **Unnecessary type conversions?** Push conversions to method arguments or callers.
6. **Easily testable?** Can you test it without heavy mocking? If not, can you test it in an integration test?
7. **Hidden dependencies?** Are there implicit dependencies that should be explicit arguments?
8. **Is the name the best option?** Brainstorm 3 alternatives:
   - Use `?` suffix for predicate methods (`published?`, `valid?`)
   - Use `!` suffix for dangerous/mutating methods (`publish!`, `save!`)
   - Use verbs for actions (`generate_playlist`, `calculate_duration`)

**Remember:** Don't refactor unless there's a compelling reason (reuse, testability, readability).

---

## Writing Quality Tests Checklist

When evaluating a test you've written:

1. **No magic literals** - Use `let`, factories, or named constants instead of `42`, `"foo"`, etc.
2. **Can it fail for real defects?** Trivial assertions like `expect(2).to eq(2)` are forbidden.
3. **Description matches assertion?** The `it` string should describe the final `expect`.
4. **Independent oracle?** Don't compare output to itself (e.g., `expect(result).to eq(method_under_test)`).
5. **Follows RuboCop rules** - Tests are code too.
6. **Tests invariants/properties?** Consider property-based testing for algorithms.
7. **Grouped logically?** Use `describe` and `context` to organize related tests.
8. **Strong assertions** - Use `eq` over `be >=`, `include` over `match(/regex/)` when possible.
9. **Tests edge cases** - Zero, negative, nil, empty, boundary values.
10. **Doesn't test types** - Ruby is dynamically typed; test behavior, not types (unless using Sorbet).

---

## Code Organization Reference

```
app/
  controllers/
    application_controller.rb           # Base controller
    music_controller.rb                 # Public music browsing
    playlists_controller.rb             # User playlists
    admin/                              # Admin namespace
      songs_controller.rb
      albums_controller.rb
  models/
    song.rb                             # Core models
    album.rb
    playlist.rb
    concerns/                           # Shared behavior
      publishable.rb
  services/                             # Business logic
    ai_conversation_service.rb
    playlist_generator.rb
  queries/                              # Complex queries
    smart_playlist_query.rb
  jobs/                                 # Background jobs
    audio_analysis_job.rb
  views/
    music/                              # Public views
      index.html.erb
    admin/                              # Admin views
      songs/
        index.html.erb
        _form.html.erb
  javascript/
    controllers/
      music/                            # Feature-specific controllers
        player_controller.js
        playlist_controller.js
      admin/
        upload_controller.js
  helpers/
    application_helper.rb               # View helpers
    music_helper.rb
```

---

## Remember Shortcuts (Optional)

The following shortcuts can be invoked at any time to trigger specific workflows.

### QNEW

When you type "qnew", this means:

```
Understand all BEST PRACTICES listed in RAILS_GUIDELINES.md.
Your code SHOULD ALWAYS follow these best practices.
```

### QPLAN

When you type "qplan", this means:

```
Analyze similar parts of the codebase and determine whether your plan:
- Is consistent with rest of codebase
- Introduces minimal changes
- Reuses existing code
- Follows patterns from RAILS_GUIDELINES.md Architecture Patterns section
```

### QCODE

When you type "qcode", this means:

```
Implement your plan and make sure your new tests pass.
Always run tests to make sure you didn't break anything else.
Always run `bundle exec rubocop -a` on newly created files to ensure standard formatting.
Always run `bundle exec rspec` to make sure tests pass.
Always run `bundle exec brakeman` to ensure no security issues.
```

### QCHECK

When you type "qcheck", this means:

```
You are a SKEPTICAL senior software engineer.
Perform this analysis for every MAJOR code change you introduced (skip minor changes):

1. RAILS_GUIDELINES.md "Writing Quality Functions/Methods Checklist"
2. RAILS_GUIDELINES.md "Writing Quality Tests Checklist"
3. RAILS_GUIDELINES.md Section 2 "While Coding" best practices
```

### QCHECKF

When you type "qcheckf", this means:

```
You are a SKEPTICAL senior software engineer.
Perform this analysis for every MAJOR function/method you added or edited (skip minor changes):

1. RAILS_GUIDELINES.md "Writing Quality Functions/Methods Checklist"
```

### QCHECKT

When you type "qcheckt", this means:

```
You are a SKEPTICAL senior software engineer.
Perform this analysis for every MAJOR test you added or edited (skip minor changes):

1. RAILS_GUIDELINES.md "Writing Quality Tests Checklist"
```

### QUX

When you type "qux", this means:

```
Imagine you are a human UX tester of the feature you implemented.
Output a comprehensive list of scenarios you would test, sorted by highest priority.
Consider edge cases, mobile responsiveness, accessibility, and user workflows.
```

### QGIT

When you type "qgit", this means:

```
Add all changes to staging, create a commit, and push to remote.
Update CLAUDE.md before every git commit.

Follow this checklist for writing your commit message:
- MUST use Conventional Commits format: https://www.conventionalcommits.org/en/v1.0.0
- MUST NOT refer to Claude or Anthropic in the commit message.
- MUST structure commit message as follows:

<type>[optional scope]: <description>

[optional body]

🤖 Generated with Claude Code assistance.

Authored-By: Mason <rogue.media.lab@gmail.com>
Co-Authored-By: Claude <noreply@anthropic.com>

Commit types (this correlates with Semantic Versioning):
- feat: introduces a new feature (MINOR version)
- fix: patches a bug (PATCH version)
- BREAKING CHANGE: breaking API change (MAJOR version, add ! after type/scope)
- Other types: build:, chore:, ci:, docs:, style:, refactor:, perf:, test:

Example:
feat(soundscape): add recently played section

- Create PlayHistory model with user associations
- Add Stimulus controller for tracking plays
- Include Turbo Stream updates for real-time display

🤖 Generated with Claude Code assistance.

Authored-By: Mason <rogue.media.lab@gmail.com>
Co-Authored-By: Claude <noreply@anthropic.com>
```

---

## Final Notes

- **Prefer convention over configuration** - Rails magic is your friend
- **Don't fight the framework** - If something feels hard, you're probably doing it wrong
- **Ship early, refactor later** - Get it working, then make it better
- **Test what matters** - Don't test Rails itself, test your business logic
- **Keep it simple** - The best code is no code

---

---

## CI Troubleshooting

### GitHub Actions: lint (RuboCop)

All RML Rails projects use RuboCop in CI via `bin/rubocop -f github`.

**Common offenses and fixes:**

- **Layout/SpaceInsideArrayLiteralBrackets**: Project rubocop config prefers spaces inside brackets. Use `[ :show, :edit ]` not `[:show, :edit]`. Applies to `before_action`, routes, `params.expect`, etc.
- **Layout/TrailingEmptyLines**: Every file must end with exactly one newline.
- **Layout/TrailingWhitespace**: No trailing spaces on any line.
- **Layout/EmptyLinesAroundClassBody**: No extra blank lines at class start or end.

**Autocorrect all offenses:**
```bash
bin/rubocop -A
```

**Verify clean before pushing:**
```bash
bin/rubocop  # exit code 0 = clean
```

### GitHub Actions: scan_ruby (Brakeman)

Brakeman runs `bin/brakeman --no-pager` and exits non-zero on errors or warnings.

**Known ERB parse error — nested escaped quotes:**

Brakeman's Ruby parser can choke on escaped quotes inside ERB double-quoted strings:

```erb
<%# BAD — causes Brakeman parse error %>
<%= button_to "Delete", path, data: { turbo_confirm: "Delete \"#{record.title}\"?" } %>

<%# GOOD — use string concatenation with single-quoted outer string %>
<% title = record.title %>
<%= button_to "Delete", path, data: { turbo_confirm: 'Delete "' + title + '"?' } %>
```

**Verify before pushing:**
```bash
bin/brakeman --no-pager  # exit code 0 = clean, 0 warnings
```

### Running both checks locally before push:
```bash
bin/rubocop && bin/brakeman --no-pager && echo "CI checks pass"
```

---

**Questions? Unclear guidelines?** Update this document as the team discovers better patterns.
