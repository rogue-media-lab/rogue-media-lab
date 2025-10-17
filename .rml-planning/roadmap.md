# Rogue Media Lab - Development Roadmap

Based on context from: claude answers.md, Studio Plan.md, and current codebase state

## Current Phase: Foundation & Community Building (Phase 1)

Goal: Build core community and deliver Soundscape MVP to first 50-100 patrons

---

## IMMEDIATE PRIORITIES (Next 2-4 Weeks)

### 1. Content (Highest Priority - Revenue Driver)

**Substack Article Completion**
- [ ] Finish "Breaking down the Monolith" paid article
  - Add refactored code examples
  - Show component-based architecture
  - Explain custom event coordination
  - Write clear takeaways section
  - Add screenshots of player
- [ ] Alternative: Consider saving this as 2nd/3rd paid article
- [ ] Option: Write broader-appeal first paid article:
  - "Building in Public: The First Month"
  - "Why I'm Building a Music Player in 2025"
  - "The Soundscape Design Philosophy"

**Status:** 19 articles published, paid tier active, writing first paid piece

### 2. Portfolio App Rebranding (Medium Priority)

**MILK-00 → Rogue Media Lab Transition**
- [ ] Update branding throughout app
  - Homepage hero section
  - Navigation header/footer
  - About page content
  - Meta tags and SEO
- [ ] Create "About Rogue Media Lab" section
  - Studio mission statement
  - Phase roadmap visualization
  - Current projects overview
  - Link to Patreon/Substack
- [ ] Polish existing sub-projects
  - Salt and Tar (mostly complete, needs minor polish)
  - Hermit Plus (landing page only, clearly mark as "Coming Soon")
  - Zuke (remove or rebrand as Soundscape preview?)
  - Blog integration (Substack embed or feed?)
- [ ] Add Stripe integration for direct support
- [ ] Analytics setup (Google Analytics or Plausible)

**Status:** Live on Heroku previously, needs redeployment with rebrand

### 3. RML Soundscape MVP Completion (High Priority - Flagship)

**Get Latest Code**
- [ ] Pull most recent code from GitHub to replace current version in repo
- [ ] Document current state vs MVP requirements

**MVP Features Audit**
- [ ] Upload songs ✓ (working)
- [ ] Play songs with waveform ✓ (working)
- [ ] Banner images/videos ✓ (working)
- [ ] Basic controls ✓ (working)
- [ ] Playlists (clunky, needs review)
- [ ] Mobile responsiveness (works but not optimized)
- [ ] Swipe gestures (fussy, needs fixing)

**Priority Fixes**
1. [ ] Playlist management UX improvements
2. [ ] Mobile UI optimization (not separate app, just responsive refinement)
3. [ ] Swipe gesture reliability
4. [ ] Admin dashboard for easier song management
5. [ ] Export/import playlist functionality

**Status:** Working locally and on Heroku, needs polish for patron release

---

## SHORT TERM (1-2 Months)

### Infrastructure

**Domain & Hosting**
- [ ] Register domain (roguemedialabs.com or similar)
- [ ] Deploy Portfolio app to Heroku
  - Configure S3 for Salt and Tar videos
  - Set up custom domain
  - SSL certificate
- [ ] Deploy Soundscape to Heroku
  - Configure S3 for audio files
  - Test streaming performance
  - Set up subdomain or path (music.roguemedialabs.com or /soundscape)

**Email & Support**
- [ ] Professional email setup (support@roguemedialabs.com)
- [ ] Contact form testing and email delivery
- [ ] Auto-responder for contact inquiries

### Content Pipeline

**Substack Schedule**
- [ ] Establish publishing cadence (weekly? bi-weekly?)
- [ ] Create content calendar for next 8 weeks
- [ ] Dev log template for consistency
- [ ] Technical tutorial series (Soundscape build logs)
- [ ] Behind-the-scenes (design decisions, challenges)

**Social Media Strategy**
- [ ] Define platform focus (TikTok like Nate? Twitter/X? YouTube shorts?)
- [ ] Content repurposing workflow (Substack → social clips)
- [ ] Posting schedule
- [ ] Engagement strategy

**Visual Assets**
- [ ] Screenshot library of all projects
- [ ] Demo video of Soundscape (60-90 seconds)
- [ ] "Work in progress" clips for social
- [ ] Before/after design comparisons

### Patreon Setup

**Tier Definitions**
- [ ] Write compelling tier descriptions
- [ ] Define exclusive benefits per tier
  - R&D Tier: Early access to micro-apps
  - Executive Tier: Additional benefits
- [ ] Create welcome post for new patrons
- [ ] First exclusive micro-app concept

---

## MEDIUM TERM (2-6 Months)

### Soundscape Public Launch

- [ ] Beta testing with patrons
- [ ] Gather and implement feedback
- [ ] Polish mobile experience
- [ ] Create user onboarding flow
- [ ] Public announcement and launch
- [ ] Press outreach (indie dev blogs, music tech sites)

### RML Voyager Development

**Salt and Tar Expansion**
- [ ] More video content
- [ ] Interactive features (comments? community?)
- [ ] Booking system for day sails (future)
- [ ] Merch integration (if applicable)

**Other Sailing Creators**
- [ ] Identify additional creators to feature
- [ ] Reach out for permission/collaboration
- [ ] Build creator submission system

### Hermit Plus Planning

- [ ] Review existing NextJS code
- [ ] Decide: Rebuild in Rails or keep NextJS?
- [ ] Design refresh in Figma
- [ ] MVP feature definition
- [ ] Start active development

### Micro-Apps for Patrons

**"App Vault" Concepts**
- [ ] Brainstorm 5-10 micro-app ideas
- [ ] Build first exclusive micro-app
- [ ] Release to R&D tier
- [ ] Gather feedback, iterate

---

## LONG TERM (6-18 Months)

### Phase 2: Launch & Expansion

- [ ] Soundscape v1.0 public launch complete
- [ ] Begin Hermit Plus active development
- [ ] Grow Patreon to 50-100 patrons
- [ ] Refine monetization model
- [ ] Consider additional revenue streams (one-time purchases? courses?)

### Phase 3: Sustainability & Ecosystem

- [ ] Achieve financial self-sufficiency from Patreon + other revenue
- [ ] Launch Hermit Plus
- [ ] Explore "Sailing Hub" concept
- [ ] Build cohesive product ecosystem
- [ ] Consider: Native mobile apps? Desktop apps? Browser extensions?

---

## BLOCKERS & DECISIONS NEEDED

### Critical Decisions

1. **First Paid Article:**
   - Complete "Breaking down Monolith" (technical, narrower audience)
   - OR write broader-appeal piece (business/philosophy, wider audience)
   - **Recommendation:** Decide by end of week

2. **Portfolio Rebrand Scope:**
   - Full redesign (2-3 weeks) or polish + content updates (3-5 days)?
   - **Recommendation:** Polish + rebrand content, save redesign for Phase 2

3. **Soundscape Latest Code:**
   - Get current version from GitHub to repo for accurate planning
   - **Recommendation:** Do this ASAP to assess true MVP status

4. **Domain Name:**
   - Need to register before deployment
   - **Recommendation:** Register this week, multiple variations

5. **Social Media Focus:**
   - Which platform(s) to prioritize for growth?
   - **Recommendation:** Research where your target audience hangs out

### Technical Debt

- [ ] Get most current Soundscape code
- [ ] Audit S3 bucket organization (separate or shared?)
- [ ] Test deployment pipelines for both apps
- [ ] Review and update dependencies (security)
- [ ] Set up staging environments

---

## SUCCESS METRICS

### Phase 1 Goals (Current)

- [ ] 50-100 Patreon patrons
- [ ] Soundscape MVP released to patrons
- [ ] Weekly Substack publishing cadence
- [ ] 1-2 micro-apps in "App Vault"
- [ ] Active social media presence (consistent posting)

### Key Metrics to Track

- Patreon subscribers (goal: 50 by end of Phase 1)
- Substack subscribers (free + paid)
- Website traffic (once deployed)
- Social media engagement
- Soundscape active users (patron beta)
- Article performance (views, conversions)

---

## NOTES

- **Strength:** Clear vision, working code, authentic voice, transparent process
- **Opportunity:** Marketing and community building need focus
- **Challenge:** Time management (automotive tech by day, builder by night)
- **Advantage:** "Build in public" resonates with indie dev community

**Next Review:** Update this roadmap monthly or after major milestones
