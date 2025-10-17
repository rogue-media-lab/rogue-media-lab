# 🎯 SalesGym MVP Plan

**Project:** AI-Powered Sales Training Platform
**Target Market:** Flooring sales teams (expandable to all home improvement)
**Tech Stack:** Rails 8 + PostgreSQL + Tailwind + Claude API
**Timeline:** 4 weeks to beta launch

---

## 🎨 PROJECT VISION

> **SalesGym transforms sales training from expensive, inconsistent, intimidating one-on-one sessions into unlimited, private, AI-powered practice with instant feedback and data-driven improvement.**

### The Problem
- Live role-play is expensive and doesn't scale
- Salespeople are intimidated to practice in front of managers
- Feedback is subjective and inconsistent
- No way to track improvement over time
- Training materials sit unused

### The Solution
- AI plays realistic homeowner personas 24/7
- Private practice environment (no judgment)
- Instant, objective scoring and feedback
- Track progress and identify weak points
- Training materials automatically recommended

---

## 🚀 MVP FEATURE SET

### **Core User Roles**

1. **Salesperson** - Practices sales conversations, receives feedback
2. **Admin** - Creates scenarios, reviews team performance

### **P0 Features (Must Have for MVP)**

#### **1. Authentication & Authorization**
- [ ] Devise for email/password login
- [ ] OmniAuth for Microsoft Azure AD (company SSO)
- [ ] Role-based access (Admin, Salesperson)
- [ ] User profiles (name, role, company)

#### **2. Scenario Management (Admin)**
- [ ] Create scenarios with:
  - Title (e.g., "Budget-Conscious DIYer")
  - Persona description (age, personality, concerns)
  - Context (project type, budget, timeline)
  - Difficulty level (Beginner, Intermediate, Advanced)
  - Success criteria (keywords/topics to cover)
- [ ] List view of all scenarios
- [ ] Edit/delete scenarios
- [ ] Activate/deactivate scenarios

#### **3. Training Material Library (Admin)**
- [ ] Upload PDFs (product guides, scripts, objection handling)
- [ ] Add external links (videos, articles)
- [ ] Tag materials (warranty, pricing, installation, objections)
- [ ] Link materials to specific scenarios or topics

#### **4. Training Session (Salesperson)**
- [ ] Browse available scenarios
- [ ] Start new training session
- [ ] Real-time chat interface
  - AI plays homeowner persona
  - Salesperson types responses
  - Conversation flows naturally
- [ ] "End Session" button
- [ ] View conversation history

#### **5. AI Conversation Engine**
- [ ] Anthropic Claude API integration
- [ ] System prompt generation from scenario data
- [ ] Maintain context throughout conversation
- [ ] Stay in character as homeowner
- [ ] Natural objections and responses
- [ ] Realistic persona behavior

#### **6. Scoring & Feedback System**
- [ ] AI analyzes completed conversation
- [ ] Generate structured feedback:
  - Overall score (0-100)
  - Strengths (array of positive actions)
  - Weaknesses (areas for improvement)
  - Keywords covered (from success criteria)
  - Would AI buy? (yes/no with reasoning)
  - Discount needed to close (0-50%)
  - Training material recommendations
- [ ] Display feedback to salesperson
- [ ] Store results in database

#### **7. Salesperson Dashboard**
- [ ] View available scenarios
- [ ] See past session history with scores
- [ ] Track score improvement over time
- [ ] Quick links to recommended training materials

#### **8. Admin Dashboard**
- [ ] Team overview (total sessions, average scores)
- [ ] Individual salesperson performance
- [ ] Scenario difficulty vs success rate
- [ ] Common weak points across team
- [ ] Session history for all users

---

## 📊 DATABASE SCHEMA

### **Users Table**
```ruby
create_table :users do |t|
  t.string :email, null: false, index: { unique: true }
  t.string :encrypted_password, null: false
  t.string :name, null: false
  t.integer :role, default: 0 # enum: salesperson, admin
  t.string :provider # for OAuth (microsoft_graph)
  t.string :uid # OAuth user ID
  t.string :company_name
  t.timestamps
end
```

### **Scenarios Table**
```ruby
create_table :scenarios do |t|
  t.string :title, null: false
  t.text :persona_description # "32-year-old first-time homeowner..."
  t.text :context # "Kitchen renovation, $10k budget..."
  t.integer :difficulty_level, default: 0 # enum: beginner, intermediate, advanced
  t.jsonb :success_criteria # { keywords: [], topics: [], objections: [] }
  t.boolean :active, default: true
  t.references :created_by, foreign_key: { to_table: :users }
  t.timestamps
end
```

### **Training Materials Table**
```ruby
create_table :training_materials do |t|
  t.string :title, null: false
  t.text :description
  t.string :material_type # enum: pdf, video, link, document
  t.string :url # For external links
  t.jsonb :tags, default: [] # ["warranty", "pricing", "installation"]
  t.references :scenario, null: true, foreign_key: true # Optional: link to specific scenario
  t.timestamps
end

# Active Storage attachment for PDFs
has_one_attached :file
```

### **Training Sessions Table**
```ruby
create_table :training_sessions do |t|
  t.references :user, null: false, foreign_key: true
  t.references :scenario, null: false, foreign_key: true
  t.integer :status, default: 0 # enum: in_progress, completed, abandoned
  t.integer :overall_score # 0-100
  t.jsonb :feedback # { strengths: [], weaknesses: [], training_links: [] }
  t.boolean :would_ai_buy
  t.integer :discount_needed # 0-50 percentage
  t.datetime :started_at
  t.datetime :ended_at
  t.integer :message_count, default: 0
  t.timestamps
end
```

### **Messages Table**
```ruby
create_table :messages do |t|
  t.references :training_session, null: false, foreign_key: true
  t.integer :role, null: false # enum: user, assistant
  t.text :content, null: false
  t.integer :sequence_number, null: false # Order in conversation
  t.timestamps
end

add_index :messages, [:training_session_id, :sequence_number]
```

---

## 🏗️ TECH ARCHITECTURE

### **Backend Services**

#### **1. AI Conversation Service**
```ruby
# app/services/ai_conversation_service.rb
class AiConversationService
  def initialize(training_session)
    @session = training_session
    @scenario = training_session.scenario
  end

  def send_message(user_input)
    # Save user message
    # Build conversation history
    # Call Claude API
    # Save AI response
    # Return response
  end

  private

  def build_system_prompt
    # Generate from scenario persona, context, difficulty
  end

  def call_claude_api(messages)
    # HTTP request to Anthropic API
  end
end
```

#### **2. Scoring Service**
```ruby
# app/services/scoring_service.rb
class ScoringService
  def score(training_session)
    # Get full conversation
    # Build scoring prompt with success criteria
    # Call Claude API for analysis
    # Parse JSON response
    # Update session with scores and feedback
  end

  private

  def build_scoring_prompt(conversation, scenario)
    # Structured prompt for AI to analyze performance
  end
end
```

### **Frontend Stack**

#### **Hotwire (Turbo + Stimulus)**
- **Turbo Frames**: Chat interface updates without full page reload
- **Turbo Streams**: Real-time message appending
- **Stimulus Controllers**:
  - `chat_controller.js` - Handle message input, auto-scroll
  - `typing_indicator_controller.js` - Show "AI is typing..."
  - `session_controller.js` - Manage session state (end, abandon)
  - `chart_controller.js` - Progress charts on dashboard

#### **Tailwind CSS**
- Clean, professional UI
- Chat bubbles (user vs AI)
- Dashboard cards and metrics
- Responsive mobile design

---

## 🗓️ 4-WEEK MVP TIMELINE

### **Week 1: Foundation & Authentication**

**Day 1-2: Rails Setup**
- [ ] Create Rails 8 app with PostgreSQL
- [ ] Set up Tailwind CSS
- [ ] Configure Devise
- [ ] Create User model with roles

**Day 3-4: Microsoft OAuth**
- [ ] Install OmniAuth gems
- [ ] Configure Azure AD integration
- [ ] Create OAuth callback controller
- [ ] Test SSO login flow

**Day 5-7: Core Models**
- [ ] Create Scenario model & migrations
- [ ] Create TrainingMaterial model with Active Storage
- [ ] Create TrainingSession model
- [ ] Create Message model
- [ ] Seed database with sample scenario

**📊 Week 1 Deliverable:** Working authentication (email + Microsoft SSO), database ready

---

### **Week 2: AI Integration & Chat**

**Day 1-2: Anthropic Claude Setup**
- [ ] Sign up for Anthropic account
- [ ] Get API key, add to credentials
- [ ] Create AiConversationService
- [ ] Test basic API calls in console

**Day 3-5: Chat Interface**
- [ ] Build chat UI with Turbo Frames
- [ ] Create Stimulus chat controller
- [ ] Implement message sending/receiving
- [ ] Add auto-scroll to latest message
- [ ] Add "AI is typing" indicator

**Day 6-7: Conversation Flow**
- [ ] Generate system prompt from scenario
- [ ] Maintain conversation context
- [ ] Handle errors gracefully
- [ ] Add "End Session" functionality
- [ ] Test full conversation flow

**📊 Week 2 Deliverable:** Working AI chat where salesperson can have full conversation with AI homeowner

---

### **Week 3: Scoring & Feedback**

**Day 1-3: Scoring System**
- [ ] Create ScoringService
- [ ] Build scoring prompt template
- [ ] Call Claude API for analysis
- [ ] Parse JSON feedback
- [ ] Store results in database

**Day 4-5: Feedback Display**
- [ ] Design feedback UI (score, strengths, weaknesses)
- [ ] Show training material recommendations
- [ ] Display "Would AI buy?" verdict
- [ ] Add discount analysis

**Day 6-7: Salesperson Dashboard**
- [ ] Scenario selection page
- [ ] Session history with scores
- [ ] Progress chart (scores over time)
- [ ] Quick stats (average score, sessions completed)

**📊 Week 3 Deliverable:** Full feedback loop - practice → scoring → improvement tracking

---

### **Week 4: Admin Tools & Polish**

**Day 1-3: Admin Dashboard**
- [ ] Team overview page
- [ ] Individual salesperson reports
- [ ] Scenario performance metrics
- [ ] Session detail view for review

**Day 4-5: Scenario Management**
- [ ] Create/edit/delete scenarios UI
- [ ] Form for persona, context, success criteria
- [ ] Scenario activation toggle
- [ ] Training material upload & tagging

**Day 6-7: Polish & Testing**
- [ ] Error handling across app
- [ ] Loading states for AI responses
- [ ] Mobile responsive testing
- [ ] Seed realistic demo data
- [ ] Write basic tests
- [ ] Deploy to staging

**📊 Week 4 Deliverable:** Fully functional MVP ready for beta testing

---

## 🎨 UI/UX DESIGN NOTES

### **Color Palette (Professional B2B)**
```
Primary: #2563eb (Blue - trust, professional)
Secondary: #10b981 (Green - growth, success)
Accent: #f59e0b (Orange - attention, warning)
Background: #f9fafb (Light gray)
Text: #111827 (Dark gray)
Error: #ef4444 (Red)
```

### **Key Pages**

#### **Salesperson Dashboard**
```
+----------------------------------+
| SalesGym Logo    |  John Doe  ▼  |
+----------------------------------+
| Your Progress                    |
| ⭐ Average Score: 78/100         |
| 📊 Sessions Completed: 12        |
| 📈 Improvement: +15 points       |
+----------------------------------+
| Available Scenarios              |
| [Card] Budget-Conscious DIYer    |
|   Difficulty: ⚡ Beginner        |
|   [Start Training]               |
|                                  |
| [Card] Skeptical Contractor      |
|   Difficulty: ⚡⚡ Intermediate   |
|   [Start Training]               |
+----------------------------------+
| Recent Sessions                  |
| Oct 15 - Excited Homeowner - 85  |
| Oct 14 - Price-Sensitive - 72    |
+----------------------------------+
```

#### **Chat Interface**
```
+----------------------------------+
| ← Back    Excited First Homeowner |
+----------------------------------+
| [AI Avatar]                      |
| Hi! Thanks for coming out...     |
|                            10:32 |
+----------------------------------+
|                   [Your Avatar]  |
|        Thanks for having me! ... |
|                 10:33            |
+----------------------------------+
| [AI Avatar]                      |
| AI is typing...                  |
+----------------------------------+
| Type your message...        [→] |
+----------------------------------+
| [End Session]                    |
+----------------------------------+
```

#### **Feedback Screen**
```
+----------------------------------+
| Session Complete! 🎉             |
+----------------------------------+
| Overall Score: 82/100            |
| ⭐⭐⭐⭐☆                          |
+----------------------------------+
| ✅ Strengths                     |
| • Asked great discovery questions|
| • Handled price objection well   |
| • Built good rapport             |
+----------------------------------+
| ⚠️ Areas to Improve              |
| • Didn't mention warranty        |
| • Rushed to close too quickly    |
+----------------------------------+
| 📚 Recommended Training          |
| [Link] Warranty Benefits Guide   |
| [Link] Closing Techniques        |
+----------------------------------+
| 🛒 Would AI Buy?                 |
| Yes, with 10% discount           |
| Reasoning: Strong pitch but price|
| was slightly high for budget...  |
+----------------------------------+
| [Practice Again] [View Dashboard]|
+----------------------------------+
```

---

## 🔧 DEVELOPMENT SETUP

### **Environment Variables**
```bash
# .env
DATABASE_URL=postgresql://localhost/salesgym_development
ANTHROPIC_API_KEY=sk-ant-xxx...

# Microsoft OAuth (for SSO)
MICROSOFT_CLIENT_ID=xxx
MICROSOFT_CLIENT_SECRET=xxx
MICROSOFT_TENANT_ID=xxx
```

### **Gemfile Additions**
```ruby
# Authentication
gem 'devise'
gem 'omniauth-microsoft_graph' # or 'omniauth-azure-activedirectory-v2'
gem 'omniauth-rails_csrf_protection'

# API calls
gem 'faraday' # HTTP client for Claude API

# Utilities
gem 'kaminari' # Pagination
gem 'ransack' # Search/filtering (admin dashboard)
```

### **Key Commands**
```bash
# Setup
bundle install
rails db:create db:migrate db:seed

# Development
bin/dev # Runs Rails + Tailwind watch

# Testing
rails test
rails test:system

# Console (for testing AI)
rails console
```

---

## 🧪 TESTING STRATEGY

### **Manual Testing Checklist**

**Authentication:**
- [ ] Email/password login works
- [ ] Microsoft SSO login works
- [ ] Role-based access (admin can't access salesperson features)

**Scenario Management:**
- [ ] Admin can create scenario
- [ ] Scenario appears in salesperson's available list
- [ ] Can edit and delete scenarios

**Training Session:**
- [ ] Start session loads AI persona correctly
- [ ] Messages send and receive properly
- [ ] Conversation maintains context
- [ ] End session triggers scoring

**Scoring:**
- [ ] Scoring completes within reasonable time (30s)
- [ ] Feedback is relevant to conversation
- [ ] Training links are appropriate
- [ ] Score is saved to database

**Dashboard:**
- [ ] Salesperson sees their sessions only
- [ ] Admin sees all team sessions
- [ ] Charts display correctly
- [ ] Progress tracking works

### **Automated Testing (Post-MVP)**
- Unit tests for models
- Service tests for AI integration
- Controller tests for CRUD
- System tests for critical flows

---

## 💰 COST ESTIMATION (MVP)

### **Development (Your Time)**
- 4 weeks × 20 hours/week = 80 hours
- At $100/hour = $8,000 opportunity cost

### **AI API Costs (Development)**
- Anthropic Claude: ~$3 per million tokens
- Average conversation: ~5,000 tokens (back-and-forth)
- Scoring analysis: ~2,000 tokens
- Total per session: ~7,000 tokens = $0.02
- 100 test sessions = $2

### **Hosting (Heroku or similar)**
- Hobby dyno: $7/month
- PostgreSQL: $9/month
- Total: ~$16/month

### **Total MVP Cost: $20-50**

### **Ongoing Costs (Production)**
Per 100 active users (10 sessions/month each):
- AI API: 1,000 sessions × $0.02 = $20
- Hosting: Bump to standard ($25) + DB ($50) = $75
- Total: ~$100/month

**Revenue (100 users at $49/month): $4,900**
**Profit margin: >95%**

---

## 📈 SUCCESS METRICS

### **MVP Validation (Beta with Wife's Team)**
- [ ] 5+ salespeople complete 3+ sessions each
- [ ] Average score improvement of 10+ points
- [ ] 80% say feedback was helpful
- [ ] Manager sees value and would pay

### **Product-Market Fit**
- [ ] 50+ active salespeople
- [ ] 500+ training sessions completed
- [ ] 3+ paying companies
- [ ] <15% monthly churn

### **Scale Readiness**
- [ ] 100+ active users
- [ ] 5,000+ sessions completed
- [ ] 10+ paying companies
- [ ] Net Promoter Score >40

---

## 🚧 KNOWN LIMITATIONS (MVP)

### **Deferred to Post-MVP:**
- Voice-based training (text-only for now)
- Advanced analytics (basic dashboard only)
- Team leaderboards / gamification
- Mobile app (responsive web only)
- Multi-language support
- RAG for training material integration (manual tags for MVP)
- Scenario difficulty auto-adjustment
- Custom AI voice/persona training

### **MVP Constraints:**
- Single company/tenant (multi-tenant in v2)
- English only
- Text conversations only (no voice)
- Manual training material tagging
- Basic reporting (no trend analysis yet)

---

## 🎯 GO-TO-MARKET STRATEGY

### **Phase 1: Beta (Weeks 1-4)**
- Your wife's flooring company
- Free access for first month
- Gather feedback intensely
- Iterate based on usage

### **Phase 2: Launch (Weeks 5-8)**
- Convert beta company to paid ($49/user/month)
- Create case study with results
- Reach out to 10 similar-sized flooring companies
- Offer 2-week free trial

### **Phase 3: Scale (Months 3-6)**
- Target 100 users across 5 companies
- Build marketing site
- Content marketing (sales training tips)
- Expand to other home improvement verticals

### **Phase 4: Platform (Months 7-12)**
- Multi-tenant SaaS
- Self-service onboarding
- Marketplace for scenario templates
- Partner program (scenario creators)

---

## 💡 FUTURE ENHANCEMENTS

### **Voice Training (High Impact)**
- Speech-to-text for user input
- Text-to-speech for AI responses
- More realistic practice experience
- Pricing: +$20/month per user

### **Advanced Analytics (High Value)**
- Weak point identification across team
- Scenario difficulty auto-tuning
- Predictive performance modeling
- Coaching recommendations for managers

### **Industry Templates (Scalability)**
- Pre-built scenarios for different industries
- Window sales, HVAC, solar, cars, insurance
- Marketplace for custom scenarios
- White-label for enterprise

### **Integration Ecosystem (Stickiness)**
- CRM integration (Salesforce, HubSpot)
- Learning Management System (LMS) integration
- Slack/Teams notifications
- Calendar integration for scheduled practice

---

## 🎬 NEXT ACTIONS

### **Immediate (Today):**
1. ✅ Create SalesGym folder
2. ⏳ You: Spin up Rails app with your template
3. ⏳ Me: Build detailed technical implementation guide

### **This Week:**
4. [ ] Set up Devise + Microsoft OAuth
5. [ ] Create database schema
6. [ ] Get Anthropic API key
7. [ ] Build first prototype chat interface

### **Week 1 Goal:**
Working login → create scenario → start training session → AI responds

---

## 📞 VALIDATION QUESTIONS (Ask Manager/Team)

Before building, validate assumptions:

1. **"Would a tool that lets salespeople practice with AI homeowners 24/7 help our team?"**
   - Expected: Yes, absolutely

2. **"What's the biggest challenge for new salespeople?"**
   - Expected: Handling objections, remembering the script, confidence

3. **"Would you pay $49/month per salesperson for unlimited AI training?"**
   - Expected: Need to see it first, but maybe

4. **"What would make this a no-brainer purchase for you?"**
   - Expected: Proof it improves close rates, easy to use, tracks progress

5. **"Can I beta test with 5 of your salespeople for free?"**
   - Expected: Yes (get your beta users!)

---

**The MVP is clear. The path is laid out. Time to build.** 🚀

**Next: You create the Rails app, I'll prep the technical implementation guide.** Ready?
