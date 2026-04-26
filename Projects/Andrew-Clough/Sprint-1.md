     1|     1|     1|     1|# Sprint 1: Foundation + Command Center Dashboard
     2|     2|     2|     2|
     3|     3|     3|     3|**Project:** RML-Clough / clough
     4|     4|     4|     4|**Goal:** Build a campaign command center — Issues CMS polish, Volunteer management, News feeds, Events with Google Calendar sync, Yard sign tracking, and Site settings
     5|     5|     5|     5|
     6|     6|     6|     6|---
     7|     7|     7|     7|
     8|     8|     8|     8|## Tasks
     9|     9|     9|     9|### 1. Setup
    10|    10|    10|    10|- [x] Apply blog template from rails-templates repo (for NewsArticle scaffold)
    11|    11|    11|    11|- [x] Generate authentication with `bin/rails generate authentication`
    12|    12|    12|    12|- [x] Create admin namespace with authenticated routes
    13|    13|    13|    13|- [x] Setup site settings model (logo, site_name, contact_email, social links, google_calendar_id)
    14|    14|    14|    14|- [x] Create Page model for static content (About, Privacy Policy)
    15|    15|    15|    15|- [ ] Create dashboard layout with navigation and stats widgets
    16|    16|    16|    16|
    17|    17|    17|    17|### 2. Issue CMS (Main Dynamic Content)
    18|    18|    18|    18|- [x] Generate Issue model + migration
    19|    19|    19|    19|  - title, description, icon, status (draft/active), position:integer, featured:boolean
    20|    20|    20|    20|  - enum + presence validations added
    21|    21|    21|    21|- [x] CRUD for Issues in admin
    22|    22|    22|    22|- [x] Dashboard view showing draft vs active issues with cards
    23|    23|    23|    23|- [x] Public home page pulling featured/active issues
    24|    24|    24|    24|- [x] Issue show page — clicking an issue card opens full article view
    25|    25|    25|    25|  - Added tagline (quote) and summary (Platform at a Glance) fields
    26|    26|    26|    26|  - Sidebar shows when summary content exists
    27|    27|    27|    27|- [x] Add WYSIWYG editor for creating/editing issues (rich text, bullets, links)
    28|    28|    28|    28|- [x] Add tagline and summary fields to issues (migration + admin form)
    29|    29|    29|    29|
    30|    30|    30|    30|### 3. Public Pages & Home Page Design
    31|    31|    31|    31|- [x] Create HomeController with index, volunteer, about, events actions
    32|    32|    32|    32|- [x] Update routes - root now points to home#index
    33|    33|    33|    33|- [x] Build full home page matching Paper design:
    34|    34|    34|    34|    - Hero section with Andrew's name and tagline
    35|    35|    35|    35|    - Meet Andrew bio section
    36|    36|    36|    36|    - Priority Issues grid (from DB)
    37|    37|    37|    37|    - Get Involved CTA
    38|    38|    38|    38|    - Latest News placeholder cards
    39|    39|    39|    39|    - Upcoming Events placeholder cards
    40|    40|    40|    40|    - Full footer with social links
    41|    41|    41|    41|- [x] Create volunteer page (/volunteer) matching Paper design
    42|    42|    42|    42|- [x] Create about page (/about) matching Paper design
    43|    43|    43|    43|- [x] Create events page (/events) matching Paper design
    44|    44|    44|    44|- [x] Create news page (/news) matching Paper design
    45|    45|    45|    45|- [x] Add campaign colors to Tailwind (navy, campaign-red, campaign-gold, Playfair Display font)
    46|    46|    46|    46|- [x] Skip authentication for public pages
    47|    47|    47|    47|
    48|    48|    48|    48|### 4. Volunteer System
    49|    49|    49|    49|- [x] Generate VolunteerInterest model (name — editable list)
    50|    50|    50|    50|- [x] Generate VolunteerSubmission model
    51|    51|    51|    51|  - name, email, phone, message, area_code, status, interest_ids
    52|    52|    52|    52|- [x] Public volunteer form with interest checkboxes
    53|    53|    53|    53|- [x] Admin list of submissions
    54|    54|    54|    54|- [ ] Volunteer dashboard — total signups, new this week/month, breakdown by interest type
    55|    55|    55|    55|- [ ] Filter volunteer list by interest, status, or date
    56|    56|    56|    56|- [ ] Track welcome email status (sent/not sent per volunteer)
    57|    57|    57|    57|
    58|    58|    58|    58|### 5. News System
    59|    59|    59|    59|- [ ] Generate NewsArticle model
    60|    60|    60|    60|  - title, body, external_url, image, source, published_date, status, featured
    61|    61|    61|    61|- [ ] Generate NewsFeed model (name, url, active)
    62|    62|    62|    62|- [ ] CRUD for articles in admin
    63|    63|    63|    63|- [ ] RSS feed management in Site Settings (add/remove feeds)
    64|    64|    64|    64|- [ ] Dashboard — recent headlines feed, categorized by topic
    65|    65|    65|    65|- [ ] AI assist: summarize articles, flag relevant stories (Phase 2)
    66|    66|    66|    66|
    67|    67|    67|    67|### 6. Events & Google Calendar Sync
    68|    68|    68|    68|- [ ] Google Calendar API integration (pull events from calendar)
    69|    69|    69|    69|- [ ] Dashboard — upcoming events widget, this week's schedule
    70|    70|    70|    70|- [ ] Event show page with Google Maps embed for location
    71|    71|    71|    71|- [ ] Site Setting for calendar connection string (test with Mason's calendar first)
    72|    72|    72|    72|- [ ] Sync status indicator (last synced, errors)
    73|    73|    73|    73|
    74|    74|    74|    74|### 7. Yard Sign Tracking
    75|    75|    75|    75|- [ ] QR code generation for yard signs (by region/zip code)
    76|    76|    76|    76|- [ ] Scan tracking endpoint (log scans with timestamp + location)
    77|    77|    77|    77|- [ ] Dashboard — scan counts by region (Rock Hill, York, Chester, etc.)
    78|    78|    78|    78|- [ ] Simple map or chart showing engagement by area
    79|    79|    79|    79|
    80|    80|    80|    80|---
    81|    81|    81|    81|
    82|    82|    82|    82|## Data Models Summary
    83|    83|    83|    83|
    84|    84|    84|    84|```
    85|    85|    85|    85|Issue
    86|    86|    86|    86|  - title:string
    87|    87|    87|    87|  - description:text (rich_text via Action Text)
    88|    88|    88|    88|  - tagline:text
    89|    89|    89|    89|  - summary:text
    90|    90|    90|    90|  - icon:string
    91|    91|    91|    91|  - status:enum [draft, active]
    92|    92|    92|    92|  - position:integer
    93|    93|    93|    93|  - featured:boolean
    94|    94|    94|    94|
    95|    95|    95|    95|VolunteerInterest
    96|    96|    96|    96|  - name:string
    97|    97|    97|    97|
    98|    98|    98|    98|VolunteerSubmission
    99|    99|    99|    99|  - name:string
   100|   100|   100|   100|  - email:string
   101|   101|   101|   101|  - phone:string
   102|   102|   102|   102|  - message:text
   103|   103|   103|   103|  - area_code:string
   104|   104|   104|   104|  - status:enum [new, contacted, confirmed, inactive]
   105|   105|   105|   105|  - interests:has_many VolunteerInterest
   106|   106|   106|   106|
   107|   107|   107|   107|NewsArticle
   108|   108|   108|   108|  - title:string
   109|   109|   109|   109|  - body:text
   110|   110|   110|   110|  - external_url:string (optional)
   111|   111|   111|   111|  - image:string
   112|   112|   112|   112|  - source:string
   113|   113|   113|   113|  - published_date:datetime
   114|   114|   114|   114|  - status:enum [draft, published]
   115|   115|   115|   115|  - featured:boolean
   116|   116|   116|   116|
   117|   117|   117|   117|NewsFeed
   118|   118|   118|   118|  - name:string
   119|   119|   119|   119|  - url:string
   120|   120|   120|   120|  - active:boolean
   121|   121|   121|   121|
   122|   122|   122|   122|Event
   123|   123|   123|   123|  - title:string
   124|   124|   124|   124|  - description:text
   125|   125|   125|   125|  - date:datetime
   126|   126|   126|   126|  - location:string
   127|   127|   127|   127|  - image:string
   128|   128|   128|   128|  - google_event_id:string
   129|   129|   129|   129|  - status:enum [upcoming, past]
   130|   130|   130|   130|
   131|   131|   131|   131|Page
   132|   132|   132|   132|  - title:string
   133|   133|   133|   133|  - slug:string (unique)
   134|   134|   134|   134|  - body:text
   135|   135|   135|   135|
   136|   136|   136|   136|SiteSetting
   137|   137|   137|   137|  - key:string (unique)
   138|   138|   138|   138|  - value:text
   139|   139|   139|   139|
   140|   140|   140|   140|YardSign
   141|   141|   141|   141|  - region:string (zip code or area name)
   142|   142|   142|   142|  - qr_code:string
   143|   143|   143|   143|  - scan_count:integer
   144|   144|   144|   144|  - last_scanned_at:datetime
   145|   145|   145|   145|```
   146|   146|   146|   146|
   147|   147|   147|   147|---
   148|   148|   148|   148|
   149|   149|   149|   149|## Out of Scope (Phase 2 / Later)
   150|   150|   150|   150|- AI assistant (summarize news, draft articles, volunteer outreach suggestions)
   151|   151|   151|   151|- Email integration for campaign communications
   152|   152|   152|   152|- Social media posting automation
   153|   153|   153|   153|- Advanced analytics and reporting
   154|   154|   154|   154|
   155|   155|   155|   155|---
   156|   156|   156|   156|
   157|   157|   157|   157|## Notes
   158|   158|   158|   158|- Use Rails 8 built-in auth (not Devise)
   159|   159|   159|   159|- Use `params.expect` per RAILS_GUIDELINES
   160|   160|   160|   160|- Single database for Solid Queue/Cache/Cable
   161|   161|   161|   161|- Running on port 3001 (dev)
   162|   162|   162|   162|- Branch: feature/issues-model
   163|   163|   163|   163|
   164|   164|   164|   164|---
   165|   165|   165|   165|
   166|   166|   166|   166|## Changelog
   167|   167|   167|   167|
   168|   168|   168|   168|### 2026-04-22
   169|   169|   169|   169|- Home page built matching Paper design
   170|   170|   170|   170|- Public pages: home, volunteer, about, events, news
   171|   171|   171|   171|- Navbar updated with campaign styling
   172|   172|   172|   172|- Tailwind theme extended with navy, campaign-red, campaign-gold
   173|   173|   173|   173|- Authentication skip for public routes
   174|   174|   174|   174|
   175|   175|   175|   175|### 2026-04-23
   176|   176|   176|   176|- Volunteer system complete: models, admin CRUD, public form
   177|   177|   177|   177|- Volunteer form layout fixed to match Paper design (2-column rows, checkbox grid)
   178|   178|   178|   178|
   179|   179|   179|   179|### 2026-04-23 (Dashboard Scope Update)
   180|   180|   180|   180|- Sprint updated to include Command Center Dashboard scope
   181|   181|   181|   181|- Added: Issue show page + WYSIWYG editor tasks
   182|   182|   182|   182|- Added: Volunteer dashboard (filtering, welcome email tracking)
   183|   183|   183|   183|- Added: News system (NewsArticle, NewsFeed, RSS management, dashboard)
   184|   184|   184|   184|- Added: Events & Google Calendar sync section
   185|   185|   185|   185|- Added: Yard sign QR tracking section
   186|   186|   186|   186|- Added: YardSign model to data models summary
   187|   187|   187|   187|- Moved Site Settings and Page model from "out of scope" to done
   188|   188|   188|   188|- Updated Out of Scope to Phase 2 (AI assistant, email integration, social posting, analytics)
   189|   189|   189|   189|### 2026-04-23 (Importmap & Action Text Setup)
   190|   190|   190|   190|- Aligned importmap/Stimulus setup with Portfolio pattern (Rails 8 conventions)
   191|   191|   191|   191|- Fixed Action Text/Trix integration (proper importmap pins, application.js imports)
   192|   192|   192|   192|- Fixed login redirect for admin users to go to admin dashboard
   193|   193|   193|   193|- Fixed Trix editor works on new issue form
   194|   194|   194|   194|### 2026-04-23 (Issue Detail Page)
   195|   195|   195|   195|- Added tagline and summary fields to issues (migration)
   196|   196|   196|   196|- Updated admin form with Tagline/Quote and Platform at a Glance fields
   197|   197|   197|   197|- Built public issue detail page with tagline quote section and summary sidebar
   198|   198|   198|   198|- Issues index and show pages working at /issues and /issues/:id
   199|   199|   199|   199|
   200|   200|   200|   200|### 2026-04-24
   201|   201|   201|   201|- Populated all 8 issues with content from cloughforsc5.com reference site
   202|   202|   202|   202|- Added taglines and summaries for all issues (Tax Relief, Jobs & Wages, Healthcare, Public Education, Immigration, Democracy & Representation, Infrastructure, Term Limits)
   203|   203|   203|   203|- Updated issues index to use tagline field for card excerpts
   204|   204|   204|   204|- Removed placeholder "Skills & Infrastructure" issue
   205|   205|   205|   205|- Verified design: clean card layout with 2-column grid, detail pages with tagline quotes and Platform at a Glance sidebar
   206|   206|   206|   206|
   207|   207|   207|   207|### 2026-04-24 (Mobile Responsive + Icons + Deploy)
   208|   208|   208|   208|- Rewrote all public pages (home, volunteer, about, events, issues, show_issue) with Tailwind responsive classes
   209|   209|   209|   209|- Added 8 missing Heroicons to issues_helper.rb (briefcase, heart, grad-cap, flag, check-circle, building, clock, money)
   210|   210|   210|   210|- Fixed GitHub Actions CI (RuboCop autocorrect 23 offenses, fixed Brakeman parse error in admin/posts/index)
   211|   211|   211|   211|- Created Procfile, seeded admin user on Heroku, deployed to Heroku (v11)
   212|   212|   212|   212|- Updated README with full project description, tech stack, deploy instructions, "Built With AI" section
   213|   213|   213|   213|- Fixed icon rendering bug: issues index/show were printing raw icon names instead of calling issue_icon helper
   214|   214|   214|   214|- Deployed to Heroku (v12)
   215|   215|   215|   215|
   216|   216|   216|   216|### 2026-04-26
   217|   217|   217|   217|- Ruby version mismatch resolved (`.ruby-version` updated to 3.4.1)
   218|   218|   218|   218|- Created `feature/dashboard-widgets` branch
   219|   219|   219|   219|- Added 8 realistic volunteer submissions to seeds.rb (distributed across Rock Hill/York/Chester area codes, mixed statuses: 3 pending, 2 contacted, 3 confirmed)
   220|   220|   220|   220|- Updated Volunteer Interests: Phone Banking, Canvassing, Event Support, Digital Outreach, Yard Signs, Data Entry
   221|   221|- Added volunteer filtering (status, interest, date range, welcome email)
   222|   222|- Added welcome_email_sent_at tracking to VolunteerSubmission
   223|   223|- Redesigned volunteer index and show pages to match dashboard style
- Merged feature/dashboard-widgets to main
- Dashboard: 5 stat cards, volunteer widgets, interest breakdown
- Sidebar: grouped nav (Overview, Content, People)
- Issues admin: icon dropdown, table layout, campaign gold styling
- Volunteers: filtering (status/interest/date/welcome), welcome email tracking
- Bug fixes: star contrast, position uniqueness validation, VolunteerInterest model fix
   224|   224|   221|   221|
   225|   225|   222|   222|### Next Up
   226|   226|   223|   223|- [x] ~~Fix Heroku Ruby version mismatch~~ — Resolved 2026-04-26
   227|   227|   224|   224|- [x] ~~Admin dashboard layout with sidebar nav and stats widgets~~ — Done
- [x] ~~Professional Issues admin UI with icon helper~~ — Done
   228|   228|   225|   225|- [ ] Volunteer dashboard — total signups, new this week/month, breakdown by interest type
   229|   229|   226|   226|- [ ] Filter volunteer list by interest, status, or date
   230|   230|   227|   227|- [ ] Track welcome email status (sent/not sent per volunteer)
   231|   231|   228|   228|
   232|   232|   229|
   233|   233|   230|---
   234|   234|   231|
   235|   235|   232|## Bugs to Fix
   236|   236|   233|1. **Low contrast stars** — Fixed. Changed from #D1D5DB to #9CA3AF.
   237|   237|   234|2. **Duplicate positions allowed** — Fixed. Added uniqueness validation on Issue model with clear error message.
   238|   238|   235|
   239|
   240|---
   241|
   242|## Future Improvements
   243|1. **Volunteer Welcome Email Template** — Create an email template that auto-populates with volunteer info (name, interests, area). Options: (a) quick-send button from admin that pre-fills a mailto or in-app email, or (b) auto-send welcome email immediately on form signup via Action Mailer.
   244|