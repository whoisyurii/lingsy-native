# Implementation Roadmap

## Overview

This roadmap breaks down the Lingsy MVP development into manageable phases, with estimated timelines and dependencies.

**Total Estimated Time**: 12-16 weeks (3-4 months)

---

## Phase 0: Setup & Infrastructure (Week 1-2)

### Objectives
- Set up development environment
- Configure Firebase
- Initialize RevenueCat
- Set up CI/CD

### Tasks

#### Week 1: Project Setup
- [x] Initialize Expo project with Ignite (Already done)
- [ ] Configure Firebase project (dev, staging, prod)
  - Create Firebase projects
  - Enable Authentication (Email/Password)
  - Create Firestore database
  - Set up Firebase Storage
  - Configure Firebase Analytics
  - Add Firebase SDK to app
- [ ] Set up environment variables
  - Create `.env.development`, `.env.staging`, `.env.production`
  - Add Firebase config
  - Add API keys
- [ ] Configure app identifiers
  - Bundle ID: `com.lingsy`
  - Update `app.json` and `app.config.ts`

#### Week 2: Backend & Services
- [ ] Set up Cloud Functions project
  - Initialize functions directory
  - Install dependencies
  - Configure TypeScript
  - Set up local emulators
- [ ] Configure RevenueCat
  - Create RevenueCat account
  - Set up products (subscriptions + gems)
  - Add RevenueCat SDK to app
  - Configure entitlements
- [ ] Set up Firestore security rules
  - Implement rules from `DATABASE_SCHEMA.md`
  - Test with emulator
- [ ] Initialize CI/CD
  - GitHub Actions workflow
  - EAS Build configuration
  - Automated testing setup

**Deliverables**:
- ✅ Firebase projects configured
- ✅ Cloud Functions scaffolded
- ✅ RevenueCat configured
- ✅ CI/CD pipeline working

---

## Phase 1: Core Authentication & Setup (Week 3-4)

### Objectives
- User authentication
- Basic navigation
- Theme system
- Internationalization

### Tasks

#### Week 3: Authentication
- [ ] Create authentication screens
  - Welcome screen
  - Signup screen (email/password)
  - Login screen
  - Language selection screen
- [ ] Implement Firebase Auth integration
  - Signup function
  - Login function
  - Logout function
  - Password reset
  - Email verification
- [ ] Create UserStore (MobX-State-Tree)
  - User model
  - Auth state management
  - Persistence with MMKV
- [ ] Implement auth flow
  - Protected routes
  - Auth state listener
  - Auto-login on launch

#### Week 4: Navigation & Design System
- [ ] Set up Expo Router structure
  - Auth stack: `(auth)/`
  - Main tabs: `(tabs)/`
  - Modals: `(modals)/`
- [ ] Create design system
  - Color palette
  - Typography scale
  - Spacing system
  - Component library (Button, Text, Card, etc.)
- [ ] Implement internationalization
  - Set up i18next
  - Create translation files: `en.ts`, `uk.ts`
  - Implement language switching
  - Test all screens in both languages
- [ ] Create common components
  - Button
  - Text
  - TextField
  - Screen layout
  - Header

**Deliverables**:
- ✅ Users can sign up and log in
- ✅ Language switching works
- ✅ Basic navigation in place
- ✅ Design system established

---

## Phase 2: Learning Core - Lessons & Exercises (Week 5-7)

### Objectives
- Lesson system
- Exercise types (flashcards, listening, writing)
- Progress tracking

### Tasks

#### Week 5: Lesson Infrastructure
- [ ] Create LessonStore (MobX-State-Tree)
  - Lesson model
  - Exercise model
  - Fetch lessons from Firestore
  - Cache lessons locally
- [ ] Create ProgressStore
  - User progress model
  - Track lesson completion
  - Track exercise results
  - Sync with Firestore
- [ ] Implement Learn screen (home)
  - Lesson path UI
  - Lesson cards
  - Scroll to current lesson
  - Lock/unlock logic
- [ ] Create Lesson detail screen
  - Lesson overview
  - Start lesson button
  - Prerequisites check
  - Heart check

#### Week 6: Exercise Types (Part 1)
- [ ] Create Exercise container component
  - Progress bar
  - Exit button
  - Heart display
  - Common layout
- [ ] Implement Flashcard exercise
  - FlashCard component with flip animation
  - Audio player
  - "Show Answer" button
  - Self-assessment buttons
  - XP calculation
- [ ] Implement Listening exercise
  - Audio player with controls
  - Speed control (normal/slow)
  - Multiple choice UI
  - Answer validation
  - Feedback animation

#### Week 7: Exercise Types (Part 2) & Completion
- [ ] Implement Writing exercise
  - Audio player
  - Text input
  - Answer validation (character-by-character)
  - Hint system (gem cost)
  - Retry logic
- [ ] Create Lesson Complete screen
  - Results summary
  - XP/gems earned display
  - Animated counters
  - Confetti celebration
  - Star rating
- [ ] Implement exercise flow logic
  - Navigate between exercises
  - Track answers
  - Calculate results
  - Update progress in Firestore

**Deliverables**:
- ✅ Users can view lessons
- ✅ All 3 exercise types working
- ✅ Progress saved to Firestore
- ✅ Lesson completion flow complete

---

## Phase 3: Gamification System (Week 8-9)

### Objectives
- XP and leveling
- Hearts system
- Gems
- Streaks
- Achievements

### Tasks

#### Week 8: XP, Hearts, Gems
- [ ] Create GameStore (MobX-State-Tree)
  - XP tracking
  - Level calculation
  - Heart management
  - Gem balance
  - Streak tracking
- [ ] Implement XP system
  - XP bar component
  - Level badge component
  - Level-up detection
  - Level-up celebration modal
  - XP formulas (see GAMIFICATION.md)
- [ ] Implement Hearts system
  - Heart display component
  - Heart regeneration timer
  - Lose heart on wrong answer
  - Heart refill modal
  - Premium unlimited hearts
- [ ] Implement Gems system
  - Gem counter component
  - Earn gems (lessons, achievements)
  - Spend gems (heart refill, hints)
  - Animated gem counter

#### Week 9: Streaks & Achievements
- [ ] Implement Streaks
  - Streak flame component
  - Daily lesson detection
  - Streak increment logic
  - Streak loss handling
  - Cloud Function: check streaks (scheduled)
- [ ] Create Achievement system
  - Achievement definitions in Firestore
  - Achievement progress tracking
  - Cloud Function: check achievements
  - Achievement unlock detection
  - Achievement unlock modal
- [ ] Create Achievements screen
  - Achievement grid
  - Category filter
  - Progress indicators
  - Locked/unlocked states
- [ ] Implement achievement unlocking
  - Real-time listeners
  - Celebration animations
  - Reward granting (XP, gems)

**Deliverables**:
- ✅ XP and leveling working
- ✅ Hearts regenerate and can be refilled
- ✅ Gems can be earned and spent
- ✅ Streaks tracked daily
- ✅ Achievements unlock automatically

---

## Phase 4: Social & Competition (Week 10)

### Objectives
- Leaderboards
- Statistics
- Profile

### Tasks

#### Week 10: Leaderboards & Profile
- [ ] Implement Leaderboards
  - Leaderboard screen
  - Tabs: Friends, Global
  - Leaderboard entries
  - User position indicator
  - Cloud Function: update leaderboards (hourly)
  - Cloud Function: reset weekly (Monday)
- [ ] Create Profile screen
  - Profile header (avatar, username, level)
  - Stats grid
  - Quick links (achievements, settings)
- [ ] Create Statistics screen
  - Time period selector
  - XP chart (bar graph)
  - Accuracy chart (line graph)
  - Time spent chart (area chart)
  - Stats aggregation
- [ ] Implement daily stats tracking
  - Track daily activity
  - Save to `dailyStats/` collection
  - Cloud Function: aggregate daily stats

**Deliverables**:
- ✅ Leaderboards updating hourly
- ✅ Profile shows all stats
- ✅ Statistics charts working
- ✅ Daily stats tracked

---

## Phase 5: Monetization (Week 11)

### Objectives
- Premium subscription
- Gem purchases
- Payment flows

### Tasks

#### Week 11: Payments & Subscriptions
- [ ] Set up RevenueCat products
  - Subscription: Monthly ($9.99)
  - Subscription: Annual ($59.99) with 7-day trial
  - Gems: 4 packages
  - Test in sandbox
- [ ] Create SubscriptionStore
  - Premium status
  - Check entitlements
  - Purchase functions
  - Restore purchases
- [ ] Create Premium modal
  - Benefits list
  - Plan selector
  - Purchase buttons
  - Restore purchases button
- [ ] Create Gem Shop modal
  - Gem packages display
  - Purchase buttons
  - Success/error handling
- [ ] Implement payment flows
  - Purchase subscription
  - Purchase gems
  - Handle success/failure
  - Update Firestore via webhook
- [ ] Create Cloud Function: revenuecatWebhook
  - Handle INITIAL_PURCHASE
  - Handle RENEWAL
  - Handle CANCELLATION
  - Handle NON_RENEWING_PURCHASE (gems)
  - Update user in Firestore
- [ ] Implement premium features
  - Unlimited hearts check
  - Monthly gems grant
  - Premium badge

**Deliverables**:
- ✅ Subscription purchases working
- ✅ Gem purchases working
- ✅ Webhooks updating Firestore
- ✅ Premium features enabled

---

## Phase 6: Content Creation & Polish (Week 12)

### Objectives
- Create lesson content
- Add vocabulary and audio
- Polish UI/UX
- Bug fixes

### Tasks

#### Week 12: Content & Polish
- [ ] Create lesson content
  - Write 20 beginner lessons
  - Create vocabulary items (200+ words)
  - Source/record audio files
  - Upload to Firebase Storage
  - Add to Firestore
- [ ] Create admin tool (simple)
  - Cloud Function: createLesson
  - Web interface or script to add content
  - Bulk upload support
- [ ] Polish UI/UX
  - Animation refinements
  - Loading states
  - Error states
  - Empty states
  - Accessibility improvements
- [ ] Bug fixes
  - Test all flows end-to-end
  - Fix identified issues
  - Performance optimization
  - Memory leak detection

**Deliverables**:
- ✅ 20 lessons with content
- ✅ All audio files uploaded
- ✅ UI polished and smooth
- ✅ Major bugs fixed

---

## Phase 7: Testing & Optimization (Week 13)

### Objectives
- Comprehensive testing
- Performance optimization
- Analytics setup

### Tasks

#### Week 13: Testing & Optimization
- [ ] Unit testing
  - Test stores (MobX-State-Tree)
  - Test utility functions
  - Test helpers
  - Coverage > 80%
- [ ] Component testing
  - Test critical components
  - Test user interactions
  - Test edge cases
- [ ] E2E testing with Maestro
  - Signup flow
  - Login flow
  - Complete lesson flow
  - Purchase flow (sandbox)
- [ ] Performance optimization
  - Profile app with React DevTools
  - Optimize re-renders
  - Lazy load heavy components
  - Image optimization
  - Bundle size reduction
- [ ] Set up analytics
  - Firebase Analytics events
  - Track key user actions
  - Set up funnels
  - Set up dashboards

**Deliverables**:
- ✅ Test coverage > 80%
- ✅ E2E tests passing
- ✅ App performance optimized
- ✅ Analytics tracking all events

---

## Phase 8: Beta Launch Preparation (Week 14)

### Objectives
- Prepare for beta launch
- TestFlight/Internal Testing
- Documentation

### Tasks

#### Week 14: Beta Prep
- [ ] Create app store assets
  - App icon
  - Screenshots (multiple devices)
  - Preview videos
  - App description (EN, UK)
- [ ] Privacy policy & Terms of Service
  - Draft legal documents
  - Add to website
  - Link in app
- [ ] Build production apps
  - iOS: EAS Build (production profile)
  - Android: EAS Build (production profile)
- [ ] Submit to TestFlight/Internal Testing
  - Upload to App Store Connect
  - Upload to Google Play Console
  - Add internal testers
- [ ] Documentation
  - User guide
  - FAQ
  - Support email setup
- [ ] Beta testing
  - Recruit 20-50 beta testers
  - Gather feedback
  - Fix critical issues

**Deliverables**:
- ✅ Apps in TestFlight/Internal Testing
- ✅ Beta testers giving feedback
- ✅ Critical issues documented
- ✅ App store assets ready

---

## Phase 9: Launch Preparation (Week 15)

### Objectives
- Fix beta issues
- Finalize store listings
- Prepare marketing

### Tasks

#### Week 15: Launch Prep
- [ ] Fix beta feedback issues
  - Prioritize critical bugs
  - UI/UX improvements
  - Content adjustments
- [ ] Finalize app store listings
  - App name, subtitle, description
  - Keywords for ASO
  - Screenshots and videos
  - Localized content (EN, UK)
- [ ] Set up support infrastructure
  - Support email
  - Help center/FAQ
  - In-app support chat (future)
- [ ] Marketing preparation
  - Landing page
  - Social media accounts
  - Press kit
  - Launch announcement
- [ ] Final testing
  - Full regression test
  - Payment testing (production)
  - Stress testing

**Deliverables**:
- ✅ Beta issues resolved
- ✅ App store listings complete
- ✅ Support infrastructure ready
- ✅ Marketing materials prepared

---

## Phase 10: Launch (Week 16)

### Objectives
- Public launch
- Monitor metrics
- Rapid response to issues

### Tasks

#### Week 16: Launch!
- [ ] Submit to app stores
  - iOS: Submit for review
  - Android: Submit for review
  - Wait for approval (1-3 days typically)
- [ ] Phased rollout
  - iOS: 10% → 50% → 100%
  - Android: Internal → Closed → Open testing → Production
- [ ] Monitor launch metrics
  - Crashlytics for crashes
  - Analytics for user behavior
  - Support requests
  - App store reviews
- [ ] Rapid response
  - Fix critical bugs immediately
  - Respond to reviews
  - Update app if needed
- [ ] Marketing push
  - Social media announcement
  - Email to waitlist
  - Press outreach
  - Community engagement

**Deliverables**:
- ✅ App live on App Store and Google Play
- ✅ No critical issues
- ✅ Users downloading and using app
- ✅ Positive reviews

---

## Post-Launch Roadmap

### Month 2-3: Iteration & Growth
- Analyze user data
- A/B test features
- Improve conversion funnel
- Add user-requested features
- Expand content (more lessons)
- Marketing campaigns

### Month 4-6: Feature Expansion
- Speaking practice
- More languages
- Social features (friends, challenges)
- Advanced gamification (leagues, tournaments)
- Content expansion (100+ lessons)

### Month 7-12: Scale
- Web app version
- B2B/Schools offering
- Premium features expansion
- Community features
- Multiple language support
- International expansion

---

## Team Structure (Recommended)

### Solo Developer (You)
- **Weeks 1-8**: Focus on core functionality
- **Weeks 9-12**: Gamification + content
- **Weeks 13-16**: Testing + launch

### Small Team (2-3 people)
- **Developer 1**: Core app, exercises, gamification
- **Developer 2**: Backend (Cloud Functions, Firestore)
- **Designer/Content**: UI/UX, content creation

### Ideal Team (4-5 people)
- **Lead Developer**: Architecture, core features
- **Frontend Developer**: UI/UX implementation
- **Backend Developer**: Cloud Functions, DevOps
- **Designer**: UI/UX design, animations
- **Content Creator**: Lessons, audio, translations

---

## Risk Mitigation

### Technical Risks
- **Risk**: Firebase costs too high
  - **Mitigation**: Optimize queries, cache data, monitor usage
- **Risk**: Performance issues
  - **Mitigation**: Early profiling, optimization, testing on low-end devices
- **Risk**: Payment integration issues
  - **Mitigation**: Thorough testing in sandbox, backup plan (manual processing)

### Business Risks
- **Risk**: Low user acquisition
  - **Mitigation**: Marketing plan, ASO, viral features
- **Risk**: Low conversion to premium
  - **Mitigation**: A/B testing, value optimization, pricing experimentation
- **Risk**: High churn
  - **Mitigation**: Engagement features, push notifications, streak system

### Content Risks
- **Risk**: Not enough content at launch
  - **Mitigation**: Focus on quality over quantity, 20 solid lessons > 50 mediocre
- **Risk**: Audio quality issues
  - **Mitigation**: Professional recording or high-quality TTS

---

## Success Metrics (First 3 Months)

### User Acquisition
- **Target**: 10,000 downloads
- **Metric**: Download velocity, install sources

### Engagement
- **Target**: 40% D1 retention, 20% D7, 10% D30
- **Metric**: Cohort retention, session length

### Monetization
- **Target**: 5% conversion to premium within 30 days
- **Metric**: Free → Premium conversion rate

### Quality
- **Target**: 4.5+ star rating on both stores
- **Metric**: App store ratings, crash-free rate >99%

### Learning Effectiveness
- **Target**: Average 80%+ accuracy on lessons
- **Metric**: Lesson completion rate, accuracy distribution

---

## Development Principles

1. **MVP First**: Ship core features, iterate based on data
2. **User-Centric**: Every decision based on user value
3. **Quality Over Speed**: Better to launch late with quality than early with bugs
4. **Test Early, Test Often**: Catch issues before they reach users
5. **Data-Driven**: Use analytics to guide decisions
6. **Iterate Fast**: Weekly deployments, continuous improvement

---

## Next Steps

1. **Week 1**: Set up Firebase and RevenueCat
2. **Week 2**: Complete authentication flow
3. **Week 3**: Build first exercise type
4. **Week 4**: Get to a working prototype
5. **Review**: Assess progress, adjust timeline if needed

---

This roadmap is ambitious but achievable. Stay focused, ship incrementally, and gather feedback early. Good luck building Lingsy! 🚀
