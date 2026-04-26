     1|# Sprint 1: Foundation + Command Center Dashboard
     2|
     3|**Project:** RML-Clough / clough
     4|**Goal:** Build a campaign command center — Issues CMS polish, Volunteer management, News feeds, Events with Google Calendar sync, Yard sign tracking, and Site settings
     5|
     6|---
     7|
     8|## Tasks
     9|### 1. Setup
    10|- [x] Apply blog template from rails-templates repo (for NewsArticle scaffold)
    11|- [x] Generate authentication with `bin/rails generate authentication`
    12|- [x] Create admin namespace with authenticated routes
    13|- [x] Setup site settings model (logo, site_name, contact_email, social links, google_calendar_id)
    14|- [x] Create Page model for static content (About, Privacy Policy)
    15|- [ ] Create dashboard layout with navigation and stats widgets
    16|
    17|### 2. Issue CMS (Main Dynamic Content)
    18|- [x] Generate Issue model + migration
    19|  - title, description, icon, status (draft/active), position:integer, featured:boolean
    20|  - enum + presence validations added
    21|- [x] CRUD for Issues in admin
    22|- [x] Dashboard view showing draft vs active issues with cards
    23|- [x] Public home page pulling featured/active issues
    24|- [x] Issue show page — clicking an issue card opens full article view
    25|  - Added tagline (quote) and summary (Platform at a Glance) fields
    26|  - Sidebar shows when summary content exists
    27|- [x] Add WYSIWYG editor for creating/editing issues (rich text, bullets, links)
    28|- [x] Add tagline and summary fields to issues (migration + admin form)
    29|
    30|### 3. Public Pages & Home Page Design
    31|- [x] Create HomeController with index, volunteer, about, events actions
    32|- [x] Update routes - root now points to home#index
    33|- [x] Build full home page matching Paper design:
    34|    - Hero section with Andrew's name and tagline
    35|    - Meet Andrew bio section
    36|    - Priority Issues grid (from DB)
    37|    - Get Involved CTA
    38|    - Latest News placeholder cards
    39|    - Upcoming Events placeholder cards
    40|    - Full footer with social links
    41|- [x] Create volunteer page (/volunteer) matching Paper design
    42|- [x] Create about page (/about) matching Paper design
    43|- [x] Create events page (/events) matching Paper design
    44|- [x] Create news page (/news) matching Paper design
    45|- [x] Add campaign colors to Tailwind (navy, campaign-red, campaign-gold, Playfair Display font)
    46|- [x] Skip authentication for public pages
    47|
    48|### 4. Volunteer System
    49|- [x] Generate VolunteerInterest model (name — editable list)
    50|- [x] Generate VolunteerSubmission model
    51|  - name, email, phone, message, area_code, status, interest_ids
    52|- [x] Public volunteer form with interest checkboxes
    53|- [x] Admin list of submissions
    54|- [ ] Volunteer dashboard — total signups, new this week/month, breakdown by interest type
    55|- [ ] Filter volunteer list by interest, status, or date
    56|- [ ] Track welcome email status (sent/not sent per volunteer)
    57|
    58|### 5. News System
    59|- [ ] Generate NewsArticle model
    60|  - title, body, external_url, image, source, published_date, status, featured
    61|- [ ] Generate NewsFeed model (name, url, active)
    62|- [ ] CRUD for articles in admin
    63|- [ ] RSS feed management in Site Settings (add/remove feeds)
    64|- [ ] Dashboard — recent headlines feed, categorized by topic
    65|- [ ] AI assist: summarize articles, flag relevant stories (Phase 2)
    66|
    67|### 6. Events & Google Calendar Sync
    68|- [ ] Google Calendar API integration (pull events from calendar)
    69|- [ ] Dashboard — upcoming events widget, this week's schedule
    70|- [ ] Event show page with Google Maps embed for location
    71|- [ ] Site Setting for calendar connection string (test with Mason's calendar first)
    72|- [ ] Sync status indicator (last synced, errors)
    73|
    74|### 7. Yard Sign Tracking
    75|- [ ] QR code generation for yard signs (by region/zip code)
    76|- [ ] Scan tracking endpoint (log scans with timestamp + location)
    77|- [ ] Dashboard — scan counts by region (Rock Hill, York, Chester, etc.)
    78|- [ ] Simple map or chart showing engagement by area
    79|
    80|---
    81|
    82|## Data Models Summary
    83|
    84|```
    85|Issue
    86|  - title:string
    87|  - description:text (rich_text via Action Text)
    88|  - tagline:text
    89|  - summary:text
    90|  - icon:string
    91|  - status:enum [draft, active]
    92|  - position:integer
    93|  - featured:boolean
    94|
    95|VolunteerInterest
    96|  - name:string
    97|
    98|VolunteerSubmission
    99|  - name:string
   100|  - email:string
   101|  - phone:string
   102|  - message:text
   103|  - area_code:string
   104|  - status:enum [new, contacted, confirmed, inactive]
   105|  - interests:has_many VolunteerInterest
   106|
   107|NewsArticle
   108|  - title:string
   109|  - body:text
   110|  - external_url:string (optional)
   111|  - image:string
   112|  - source:string
   113|  - published_date:datetime
   114|  - status:enum [draft, published]
   115|  - featured:boolean
   116|
   117|NewsFeed
   118|  - name:string
   119|  - url:string
   120|  - active:boolean
   121|
   122|Event
   123|  - title:string
   124|  - description:text
   125|  - date:datetime
   126|  - location:string
   127|  - image:string
   128|  - google_event_id:string
   129|  - status:enum [upcoming, past]
   130|
   131|Page
   132|  - title:string
   133|  - slug:string (unique)
   134|  - body:text
   135|
   136|SiteSetting
   137|  - key:string (unique)
   138|  - value:text
   139|
   140|YardSign
   141|  - region:string (zip code or area name)
   142|  - qr_code:string
   143|  - scan_count:integer
   144|  - last_scanned_at:datetime
   145|```
   146|
   147|---
   148|
   149|## Out of Scope (Phase 2 / Later)
   150|- AI assistant (summarize news, draft articles, volunteer outreach suggestions)
   151|- Email integration for campaign communications
   152|- Social media posting automation
   153|- Advanced analytics and reporting
   154|
   155|---
   156|
   157|## Notes
   158|- Use Rails 8 built-in auth (not Devise)
   159|- Use `params.expect` per RAILS_GUIDELINES
   160|- Single database for Solid Queue/Cache/Cable
   161|- Running on port 3001 (dev)
   162|- Branch: feature/issues-model
   163|
   164|---
   165|
   166|## Changelog
   167|
   168|### 2026-04-22
   169|- Home page built matching Paper design
   170|- Public pages: home, volunteer, about, events, news
   171|- Navbar updated with campaign styling
   172|- Tailwind theme extended with navy, campaign-red, campaign-gold
   173|- Authentication skip for public routes
   174|
   175|### 2026-04-23
   176|- Volunteer system complete: models, admin CRUD, public form
   177|- Volunteer form layout fixed to match Paper design (2-column rows, checkbox grid)
   178|
   179|### 2026-04-23 (Dashboard Scope Update)
   180|- Sprint updated to include Command Center Dashboard scope
   181|- Added: Issue show page + WYSIWYG editor tasks
   182|- Added: Volunteer dashboard (filtering, welcome email tracking)
   183|- Added: News system (NewsArticle, NewsFeed, RSS management, dashboard)
   184|- Added: Events & Google Calendar sync section
   185|- Added: Yard sign QR tracking section
   186|- Added: YardSign model to data models summary
   187|- Moved Site Settings and Page model from "out of scope" to done
   188|- Updated Out of Scope to Phase 2 (AI assistant, email integration, social posting, analytics)
   189|### 2026-04-23 (Importmap & Action Text Setup)
   190|- Aligned importmap/Stimulus setup with Portfolio pattern (Rails 8 conventions)
   191|- Fixed Action Text/Trix integration (proper importmap pins, application.js imports)
   192|- Fixed login redirect for admin users to go to admin dashboard
   193|- Fixed Trix editor works on new issue form
   194|### 2026-04-23 (Issue Detail Page)
   195|- Added tagline and summary fields to issues (migration)
   196|- Updated admin form with Tagline/Quote and Platform at a Glance fields
   197|- Built public issue detail page with tagline quote section and summary sidebar
   198|- Issues index and show pages working at /issues and /issues/:id
   199|
   200|### 2026-04-24
   201|- Populated all 8 issues with content from cloughforsc5.com reference site
   202|- Added taglines and summaries for all issues (Tax Relief, Jobs & Wages, Healthcare, Public Education, Immigration, Democracy & Representation, Infrastructure, Term Limits)
   203|- Updated issues index to use tagline field for card excerpts
   204|- Removed placeholder "Skills & Infrastructure" issue
   205|- Verified design: clean card layout with 2-column grid, detail pages with tagline quotes and Platform at a Glance sidebar
   206|
   207|### 2026-04-24 (Mobile Responsive + Icons + Deploy)
   208|- Rewrote all public pages (home, volunteer, about, events, issues, show_issue) with Tailwind responsive classes
   209|- Added 8 missing Heroicons to issues_helper.rb (briefcase, heart, grad-cap, flag, check-circle, building, clock, money)
   210|- Fixed GitHub Actions CI (RuboCop autocorrect 23 offenses, fixed Brakeman parse error in admin/posts/index)
   211|- Created Procfile, seeded admin user on Heroku, deployed to Heroku (v11)
   212|- Updated README with full project description, tech stack, deploy instructions, "Built With AI" section
   213|- Fixed icon rendering bug: issues index/show were printing raw icon names instead of calling issue_icon helper
   214|- Deployed to Heroku (v12)
   215|
   216|### 2026-04-26
   217|- Ruby version mismatch resolved (`.ruby-version` updated to 3.4.1)
   218|- Created `feature/dashboard-widgets` branch
   219|- Added 8 realistic volunteer submissions to seeds.rb (distributed across Rock Hill/York/Chester area codes, mixed statuses: 3 pending, 2 contacted, 3 confirmed)
   220|- Updated Volunteer Interests: Phone Banking, Canvassing, Event Support, Digital Outreach, Yard Signs, Data Entry
   221|
   222|### Next Up
   223|- [x] ~~Fix Heroku Ruby version mismatch~~ — Resolved 2026-04-26
   224|- [ ] Admin dashboard layout with sidebar nav and stats widgets
   225|- [ ] Volunteer dashboard — total signups, new this week/month, breakdown by interest type
   226|- [ ] Filter volunteer list by interest, status, or date
   227|- [ ] Track welcome email status (sent/not sent per volunteer)
   228|

---

## Bugs to Fix
1. **Low contrast stars** — Gray outline stars (☆) for non-featured issues are hard to see. Need a darker/more visible color.
2. **Duplicate positions allowed** — Multiple issues can be assigned the same position number, causing ordering conflicts on the public site. Need to either auto-reorder siblings or show a warning when duplicates exist.
