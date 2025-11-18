# Lingsy Screens Specification

## Navigation Structure

```
App
├── (auth) - Authentication Flow
│   ├── welcome.tsx
│   ├── login.tsx
│   ├── signup.tsx
│   └── language-select.tsx (choose interface language)
│
├── (tabs) - Main App (Bottom Tabs)
│   ├── learn/
│   │   ├── index.tsx (Home/Learn)
│   │   ├── lesson/[id].tsx
│   │   ├── exercise/[type]/[id].tsx
│   │   └── lesson-complete.tsx
│   │
│   ├── practice/
│   │   ├── index.tsx (Practice modes)
│   │   └── flashcards.tsx
│   │
│   ├── leaderboard/
│   │   └── index.tsx
│   │
│   └── profile/
│       ├── index.tsx
│       ├── settings.tsx
│       ├── statistics.tsx
│       └── achievements.tsx
│
└── (modals)
    ├── shop.tsx (Gem shop)
    ├── premium.tsx (Subscription)
    ├── health.tsx (Refill hearts)
    └── achievement-unlock.tsx
```

---

## Authentication Flow Screens

### 1. Welcome Screen (`/welcome`)

**Purpose**: First screen users see, onboarding

**Components**:
- App logo and tagline
- Language selection dropdown (EN/UK interface)
- "Get Started" button
- "Already have an account?" link

**Actions**:
- Navigate to language select
- Navigate to login

**Gamification**: None

---

### 2. Language Selection Screen (`/language-select`)

**Purpose**: Choose interface language and initial learning language

**Components**:
- "Choose your language" title
- Interface language selector (EN/UK)
- "I want to learn..." section
- Language card: English (only option for MVP)
  - Flag icon
  - Language name
  - Number of learners
- Continue button

**Actions**:
- Set interface language
- Set learning language (English)
- Navigate to signup

**Gamification**: None

---

### 3. Sign Up Screen (`/signup`)

**Purpose**: Create new account

**Components**:
- Email input
- Password input
- Confirm password input
- Age confirmation checkbox
- Terms & Privacy links
- "Create Account" button
- "Already have account?" link

**Actions**:
- Create Firebase user
- Send email verification
- Navigate to main app

**Validation**:
- Valid email format
- Password min 8 characters
- Passwords match
- Age 13+

---

### 4. Login Screen (`/login`)

**Purpose**: Existing user authentication

**Components**:
- Email input
- Password input
- "Forgot password?" link
- "Log In" button
- "Don't have account?" link

**Actions**:
- Authenticate with Firebase
- Navigate to main app

---

## Main App Screens (Tabs)

### 5. Learn/Home Screen (`/(tabs)/learn/index`)

**Purpose**: Main learning hub, start lessons

**Components**:

**Header Section**:
- User avatar (top-left)
- Streak flame icon + count
- Gems count
- Hearts count (with regeneration timer if < 5)

**Progress Section**:
- Level progress bar (XP to next level)
- Current level badge
- Daily goal progress circle

**Learning Path**:
- Scrollable vertical path of lessons
- Each lesson node shows:
  - Lesson number & title
  - Lock icon (if locked)
  - Check mark (if completed)
  - Star rating (0-3 stars based on performance)
  - Current lesson highlighted with glow

**Quick Practice Section**:
- "Practice" button
- "Review Mistakes" button
- "Flashcards" button

**Bottom Tab Navigation**:
- Learn (selected)
- Practice
- Leaderboard
- Profile

**Actions**:
- Tap lesson → Navigate to lesson screen
- Tap practice → Navigate to practice modes
- Tap hearts → Open health refill modal
- Tap gems → Open gem shop

**Gamification**:
- Visual path progression
- Streak counter
- Hearts system
- Level display
- XP progress

---

### 6. Lesson Screen (`/(tabs)/learn/lesson/[id]`)

**Purpose**: Pre-lesson overview and start

**Components**:
- Lesson title
- Lesson description
- Preview of what will be learned (5-7 new words)
- Difficulty indicator
- Estimated time (5-10 min)
- Reward preview:
  - XP amount
  - Gems amount (if first time)
- "Start Lesson" button
- "Back" button

**Actions**:
- Start lesson → Navigate to first exercise
- Back → Return to learn screen

**Requirements**:
- Check if user has hearts (min 1)
- Check if previous lesson completed (if applicable)

---

### 7. Exercise Screen (`/(tabs)/learn/exercise/[type]/[id]`)

**Purpose**: Individual exercise within a lesson

**Types**: Flashcard, Listening, Writing

**Common Components (all types)**:
- Top bar:
  - Exit button (with confirmation)
  - Progress bar (exercise X of Y)
  - Hearts count
- Question area (type-specific)
- Answer area (type-specific)
- "Check" or "Continue" button
- Feedback overlay (correct/incorrect)

#### 7a. Flashcard Exercise

**Question Area**:
- Ukrainian word/phrase (large text)
- Image/icon (if available)
- Audio play button

**Answer Area**:
- Initially hidden (tap to reveal)
- Shows English translation
- "Show Answer" button
- After reveal: "I got it right" / "I got it wrong" buttons

**Feedback**:
- Confetti animation (if correct)
- Encouraging message
- Correct answer highlighted

#### 7b. Listening Exercise

**Question Area**:
- Audio play button (large, centered)
- Sound wave animation during playback
- Speed control (normal/slow)
- "Replay" button

**Answer Area**:
- 4 multiple choice options (English words)
- Each option is a button

**Feedback**:
- Correct option turns green
- Incorrect option (if selected) turns red then shows correct
- Audio auto-plays for correct answer

#### 7c. Writing Exercise (Write to Match)

**Question Area**:
- Audio play button
- Instruction: "Type what you hear"
- Replay button

**Answer Area**:
- Text input field
- Keyboard automatically shown
- Character count
- "Hint" button (costs 1 gem or shows first letter)

**Feedback**:
- Highlight incorrect characters in red
- Show correct answer if wrong
- Allow retry once

**Actions** (all exercise types):
- Submit answer
- Check if correct
- Update XP/hearts
- Navigate to next exercise or lesson complete

**Gamification**:
- XP per correct answer (+10)
- Perfect streak bonus (+5 XP per streak)
- Lose heart on wrong answer
- Combo multiplier (3+ correct in a row)

---

### 8. Lesson Complete Screen (`/(tabs)/learn/lesson-complete`)

**Purpose**: Celebrate lesson completion, show rewards

**Components**:
- Celebration animation (confetti, stars)
- "Lesson Complete!" title
- Performance summary:
  - Accuracy percentage
  - Correct/Total answers
  - Time taken
- Rewards earned:
  - XP gained (with animation)
  - Gems earned (first time only)
  - New level (if leveled up)
- New achievements (if any)
- "Continue" button
- "Review Mistakes" button (if any incorrect)

**Actions**:
- Continue → Return to learn screen
- Review → Show incorrect exercises

**Gamification**:
- XP animation counting up
- Level up animation
- Achievement popup

---

### 9. Practice Screen (`/(tabs)/practice/index`)

**Purpose**: Extra practice outside of lessons

**Components**:
- "Practice Modes" title
- Practice mode cards:

  **Flashcards**:
  - Icon
  - "Flashcards" title
  - "Review vocabulary"
  - No heart cost

  **Listening Practice**:
  - Icon
  - "Listening Practice"
  - "Sharpen your ears"
  - Costs 1 heart per session

  **Speed Review**:
  - Icon
  - "Speed Review"
  - "Quick practice"
  - Costs 1 heart per session

  **Review Mistakes**:
  - Icon
  - "Review Mistakes"
  - Shows count of mistakes to review
  - No heart cost

**Actions**:
- Tap mode → Start practice session
- Check hearts requirement

---

### 10. Flashcards Practice (`/(tabs)/practice/flashcards`)

**Purpose**: Free vocabulary review with flashcards

**Components**:
- Deck selector (All, Recent, Difficult, Favorites)
- Card counter (Card X of Y)
- Flashcard (flippable):
  - Front: Ukrainian word + image
  - Back: English translation
- Swipe gestures:
  - Swipe right: "I know this"
  - Swipe left: "Review again"
  - Swipe up: "Add to favorites"
- Audio play button
- "Shuffle" button
- Progress indicator

**Actions**:
- Flip card
- Swipe to categorize
- Mark favorites
- Audio playback

**Gamification**:
- Small XP for each card reviewed (+2 XP)
- No hearts cost

---

### 11. Leaderboard Screen (`/(tabs)/leaderboard/index`)

**Purpose**: Social competition and motivation

**Components**:
- Tab selector:
  - Friends
  - Global (country)
  - League

**Friends Tab**:
- "Add friends" button
- List of friends with:
  - Avatar
  - Name
  - XP this week
  - Position change (↑↓)

**Global Tab**:
- User's position (pinned at top if not in top 10)
- Top 50 learners
- Each entry shows:
  - Rank
  - Avatar
  - Username
  - XP this week
  - Country flag

**League Tab** (Future):
- Current league (Bronze, Silver, Gold, etc.)
- Top 10 in current league
- Promotion/demotion zone indicators
- Time until league ends

**Actions**:
- Tap user → View profile (if friend)
- Add friends button → Friend search

**Gamification**:
- Weekly XP competition
- Rank badges
- Position change indicators

---

### 12. Profile Screen (`/(tabs)/profile/index`)

**Purpose**: User profile and account management

**Components**:

**Header**:
- Large avatar (editable)
- Username
- Level badge
- Total XP
- Member since date

**Stats Section**:
- Learning streak (current + longest)
- Total lessons completed
- Accuracy rate
- Time spent learning

**Quick Actions**:
- "View Achievements" button
- "Statistics" button
- "Settings" button

**Learning Section**:
- Current language: English
- Interface language: EN/UK
- "Change languages" button (future)

**Premium Section** (if free user):
- "Upgrade to Premium" banner
- Benefits preview

**Actions**:
- Edit profile
- View detailed stats
- Manage subscription
- Open settings

---

### 13. Settings Screen (`/(tabs)/profile/settings`)

**Purpose**: App configuration and preferences

**Components**:

**Account**:
- Email (display only)
- Password (change button)
- Delete account button

**Preferences**:
- Notification settings:
  - Daily reminder toggle + time
  - Streak reminder toggle
  - Achievement alerts toggle
- Sound effects toggle
- Haptic feedback toggle
- Audio autoplay toggle

**App**:
- Interface language selector
- Download lessons for offline
- Clear cache button
- App version

**Legal**:
- Terms of Service
- Privacy Policy
- Licenses

**Subscription** (if premium):
- Plan type
- Renewal date
- Manage subscription button

**Actions**:
- Toggle settings
- Update preferences
- Manage account

---

### 14. Statistics Screen (`/(tabs)/profile/statistics`)

**Purpose**: Detailed learning analytics

**Components**:

**Time Period Selector**:
- Week / Month / Year / All Time

**Charts**:
- XP per day (bar chart)
- Lessons completed (line graph)
- Accuracy over time (line graph)
- Time spent learning (area chart)

**Stats Grid**:
- Total lessons: X
- Accuracy: Y%
- Current streak: Z days
- Longest streak: A days
- Words learned: B
- Perfect lessons: C

**Recent Activity**:
- Last 10 lessons with results

**Actions**:
- Switch time period
- View detailed lesson results

---

### 15. Achievements Screen (`/(tabs)/profile/achievements`)

**Purpose**: View and track achievements

**Components**:
- "Achievements" title
- XP earned from achievements total

**Categories** (tabs):
- All
- Learning
- Streaks
- Social
- Special

**Achievement Grid**:
Each achievement shows:
- Icon (color if unlocked, grayscale if locked)
- Title
- Description
- Progress bar (if in progress)
- XP reward
- Unlock date (if unlocked)

**Example Achievements**:

**Learning**:
- First Lesson (5 XP)
- 10 Lessons (25 XP)
- 50 Lessons (100 XP)
- 100 Lessons (250 XP)
- Perfect Lesson (10 XP)
- 10 Perfect Lessons (50 XP)

**Streaks**:
- 3 Day Streak (10 XP)
- 7 Day Streak (25 XP)
- 30 Day Streak (100 XP)
- 100 Day Streak (500 XP)
- Weekend Warrior (15 XP) - 2 consecutive weekend days

**Social**:
- Add First Friend (10 XP)
- Beat a Friend (15 XP)
- Top 10 in League (50 XP)

**Special**:
- Early Adopter (100 XP)
- Night Owl (10 XP) - Complete lesson after 10 PM
- Early Bird (10 XP) - Complete lesson before 7 AM
- Speed Demon (25 XP) - Complete lesson in under 3 min

**Actions**:
- Filter by category
- Share achievement (if unlocked)

**Gamification**:
- Visual progress tracking
- XP rewards
- Shareability

---

## Modal Screens

### 16. Gem Shop Modal (`/shop`)

**Purpose**: Purchase gems with real money

**Components**:
- "Gem Shop" title
- Close button

**Gem Packages**:
- Small: 100 gems - $0.99
- Medium: 500 gems - $3.99 (20% bonus)
- Large: 1200 gems - $7.99 (50% bonus)
- Mega: 3000 gems - $14.99 (100% bonus)

Each package shows:
- Gem icon with count
- Price
- Bonus percentage (if any)
- "Buy" button

**Restore Purchases** button

**Actions**:
- Purchase gems via RevenueCat
- Restore previous purchases
- Close modal

---

### 17. Premium Subscription Modal (`/premium`)

**Purpose**: Upsell premium subscription

**Components**:
- "Go Premium" title
- Close button (X)

**Benefits List**:
- ✓ Unlimited hearts
- ✓ No ads (future)
- ✓ Offline lessons
- ✓ Personalized learning
- ✓ Mastery quizzes
- ✓ Monthly gems (500)
- ✓ Exclusive achievements

**Pricing Options**:
- Monthly: $9.99/month
- Annual: $59.99/year (save 50%)

Each option shows:
- Price
- Billing period
- Savings (if any)
- "Subscribe" button

**"Restore Purchases"** link
**"Terms & Privacy"** link

**Actions**:
- Subscribe via RevenueCat
- Close modal

---

### 18. Health Refill Modal (`/health`)

**Purpose**: Refill hearts when depleted

**Components**:
- "Out of hearts!" title
- Heart icons showing 0/5
- Regeneration timer (e.g., "Full in 2h 30m")

**Refill Options**:
- Wait for regeneration (free)
  - Timer countdown
  - "I'll wait" button

- Watch ad for 1 heart (future)
  - "Watch Ad" button

- Refill with gems (50 gems)
  - "Refill Now" button

- Subscribe to Premium
  - "Get Unlimited" button
  - Links to premium modal

**Actions**:
- Wait (close modal)
- Watch ad
- Spend gems
- Navigate to premium

**Gamification**:
- Clear timer display
- Multiple options with tradeoffs

---

### 19. Achievement Unlock Modal (`/achievement-unlock`)

**Purpose**: Celebrate achievement unlock

**Components**:
- Confetti animation
- Achievement icon (large, glowing)
- "Achievement Unlocked!" title
- Achievement name
- Achievement description
- XP earned (animated count-up)
- "Share" button
- "Awesome!" button (close)

**Actions**:
- Share to social media
- Close modal

**Gamification**:
- Celebratory animation
- XP reward emphasis
- Social sharing

---

## Future Screens (Post-MVP)

### 20. Story Screen
- Read stories in target language
- Tap words for translation
- Audio narration
- Comprehension questions

### 21. Conversation Practice
- AI-powered conversations
- Voice input
- Real-time feedback

### 22. Grammar Lessons
- Explanation screens
- Practice exercises
- Reference guide

### 23. Social Features
- Friend profiles
- Direct challenges
- Group learning

### 24. Offline Mode Manager
- Download lessons
- Manage downloaded content
- Storage usage

---

## Screen Interactions Flow

```
Welcome
  → Language Select
    → Signup/Login
      → Learn Screen (Home)
        → Lesson Overview
          → Exercise (Flashcard/Listening/Writing)
            → Lesson Complete
              → Back to Learn Screen

Learn Screen
  → Practice → Practice Modes → Flashcards/Listening/etc
  → Leaderboard
  → Profile
    → Achievements
    → Statistics
    → Settings

Modals (can be triggered from anywhere):
  - Gem Shop (from gem icon)
  - Premium (from various prompts)
  - Health Refill (when hearts = 0)
  - Achievement Unlock (on achievement earn)
```

---

## Responsive Design Considerations

- All screens must work on iPhone SE (small) to iPad (large)
- Use relative sizing (flex, percentages)
- Safe area insets (notches, home indicators)
- Landscape support for tablets
- Font scaling for accessibility
- Minimum touch target: 44x44 points

---

## Animation & Transitions

- Screen transitions: Slide (iOS), Fade (Android)
- Modal presentation: Slide up
- Tab switching: Fade
- Exercise completion: Scale + confetti
- XP gain: Count-up animation
- Level up: Full-screen celebration
- Achievement: Pop + bounce

---

## Loading States

Every screen must have:
- Initial load skeleton
- Pull-to-refresh (where applicable)
- Error states with retry
- Empty states with CTA
- Offline indicators

---

## Accessibility

- VoiceOver/TalkBack support
- Dynamic type support
- High contrast mode
- Reduce motion option
- Minimum color contrast: 4.5:1
- Clear focus indicators
- Descriptive labels

---

This specification covers all MVP screens. Each screen should be implemented with proper error handling, loading states, and offline support where applicable.
