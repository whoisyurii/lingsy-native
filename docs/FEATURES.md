# Feature Specifications

## MVP Features (Phase 1)

### 1. User Authentication

**Description**: Secure user registration and login

**User Stories**:
- As a new user, I want to sign up with email/password
- As a returning user, I want to log in to access my progress
- As a user, I want to reset my password if I forget it

**Acceptance Criteria**:
- ✅ Email/password signup with validation
- ✅ Email verification sent
- ✅ Login with existing account
- ✅ "Forgot password" flow
- ✅ Error messages for invalid inputs
- ✅ Password strength requirements (min 8 characters)
- ✅ Auto-login on app launch (persistent session)

**Technical Implementation**:
- Firebase Authentication
- Email validation regex
- Secure password storage (handled by Firebase)
- Token-based session management
- MMKV for local auth token storage

**UI Components**:
- Welcome screen
- Signup form
- Login form
- Password reset modal

---

### 2. Interface Language Selection

**Description**: Choose UI language (English or Ukrainian)

**User Stories**:
- As a user, I want to use the app in my preferred language
- As a user, I want to change the interface language later in settings

**Acceptance Criteria**:
- ✅ Language selector on welcome screen
- ✅ Two options: English, Ukrainian
- ✅ All UI text updates immediately
- ✅ Preference saved to user profile
- ✅ Changeable in settings

**Technical Implementation**:
- i18next for internationalization
- Translation files: `en.ts`, `uk.ts`
- Context provider for language state
- Firestore: `users/{id}.interfaceLanguage`

**UI Components**:
- Language selector dropdown
- Language switch in settings

---

### 3. Learning Path (Lesson Tree)

**Description**: Visual progression through lessons

**User Stories**:
- As a learner, I want to see my learning path
- As a learner, I want to know which lessons I've completed
- As a learner, I want to know which lesson to do next

**Acceptance Criteria**:
- ✅ Vertical scrollable path of lessons
- ✅ Lessons show: number, title, status (locked/current/completed)
- ✅ Completed lessons show star rating (0-3)
- ✅ Current lesson highlighted with glow
- ✅ Locked lessons show lock icon
- ✅ Tap lesson to view details or start
- ✅ Sequential unlocking (complete previous to unlock next)

**Technical Implementation**:
- FlatList with lesson nodes
- Lessons ordered by `order` field
- Check prerequisites before allowing start
- Visual states: locked, unlocked, in-progress, completed
- Smooth scroll to current lesson on load

**UI Components**:
- LessonCard (custom component)
- LessonPath (scrollable list)
- Lock icon, checkmark icon, star icons

**Data Flow**:
```
Load lessons from Firestore
  → Sort by order
  → Get user progress for each
  → Determine locked/unlocked state
  → Render LessonCard components
```

---

### 4. Flashcard Exercises

**Description**: Vocabulary flashcards with audio

**User Stories**:
- As a learner, I want to learn new words with flashcards
- As a learner, I want to hear the pronunciation
- As a learner, I want to self-assess if I got it right

**Acceptance Criteria**:
- ✅ Front: Ukrainian word + image (if available)
- ✅ Back: English translation
- ✅ Audio play button (auto-play on reveal)
- ✅ "Show Answer" button
- ✅ After reveal: "I got it right" / "I got it wrong" buttons
- ✅ Track correct/incorrect for stats
- ✅ Smooth flip animation

**Technical Implementation**:
- FlashCard component with flip animation (Reanimated)
- Audio playback with expo-av
- Track user's self-assessment
- Update userProgress in Firestore

**UI Components**:
- FlashCard (flippable)
- Audio player button
- Self-assessment buttons

**Exercise Flow**:
```
Load exercise
  → Show front (Ukrainian)
  → User taps "Show Answer"
    → Flip animation
    → Show back (English)
    → Play audio
  → User selects "Right" or "Wrong"
    → Update progress
    → Award XP if correct
    → Load next exercise
```

---

### 5. Listening Exercises

**Description**: Listen and choose correct translation

**User Stories**:
- As a learner, I want to practice listening comprehension
- As a learner, I want to replay audio if I didn't catch it
- As a learner, I want to slow down the audio if needed

**Acceptance Criteria**:
- ✅ Audio plays automatically on load
- ✅ Large play button in center
- ✅ Sound wave animation during playback
- ✅ "Replay" button
- ✅ Speed control: Normal / Slow (0.75x)
- ✅ 4 multiple choice options
- ✅ Select answer and submit
- ✅ Immediate feedback (correct = green, incorrect = red)
- ✅ Show correct answer if wrong
- ✅ Auto-play correct answer audio

**Technical Implementation**:
- expo-av for audio playback
- Speed control: `playbackRate` property
- Multiple choice options shuffled
- Visual feedback with color change animation
- Heart deduction on wrong answer

**UI Components**:
- Large play button with animation
- Speed selector toggle
- Multiple choice buttons (4 options)
- Feedback overlay

**Exercise Flow**:
```
Load exercise
  → Auto-play audio
  → Show 4 options (shuffled)
  → User selects option
  → User taps "Check"
    → If correct:
      → Green animation + confetti
      → Award XP
    → If incorrect:
      → Red animation
      → Deduct 1 heart
      → Show correct answer (green)
  → "Continue" to next exercise
```

---

### 6. Writing Exercises (Type What You Hear)

**Description**: Listen and type the correct word/phrase

**User Stories**:
- As a learner, I want to practice writing
- As a learner, I want to type what I hear
- As a learner, I want hints if I'm stuck

**Acceptance Criteria**:
- ✅ Audio plays automatically
- ✅ "Replay" button visible
- ✅ Text input field auto-focused (keyboard appears)
- ✅ "Hint" button (costs 10 gems or shows first letter)
- ✅ "Check" button
- ✅ Character-by-character validation
- ✅ Highlight incorrect characters in red
- ✅ Allow one retry
- ✅ Show correct answer after 2 attempts

**Technical Implementation**:
- expo-av for audio playback
- TextInput with validation
- String comparison (case-insensitive)
- Optional: accept alternative answers
- Gem cost for hint

**UI Components**:
- Audio player with replay
- TextInput (custom styled)
- Hint button
- Character feedback (color-coded)

**Exercise Flow**:
```
Load exercise
  → Auto-play audio
  → Show input field (keyboard appears)
  → User types answer
  → User taps "Check"
    → Compare with correct answer
    → If correct:
      → Green checkmark + confetti
      → Award XP
      → Continue
    → If incorrect:
      → Highlight wrong characters in red
      → Allow retry (attempt 2)
      → If still wrong:
        → Show correct answer
        → Deduct 1 heart
        → Continue
```

**Hint System**:
```
User taps "Hint"
  → Modal: "Use 10 gems for a hint?"
  → If confirmed:
    → Deduct 10 gems
    → Show first letter in input
  → If not enough gems:
    → Prompt to buy gems
```

---

### 7. Lesson Completion & Results

**Description**: Summary of lesson performance with rewards

**User Stories**:
- As a learner, I want to see my lesson results
- As a learner, I want to see what I earned (XP, gems)
- As a learner, I want to review my mistakes

**Acceptance Criteria**:
- ✅ Celebration animation (confetti, stars)
- ✅ "Lesson Complete!" title
- ✅ Performance summary: Accuracy %, correct/total, time
- ✅ Rewards: XP earned (animated count-up), gems (if first-time)
- ✅ Star rating (0-3 based on accuracy)
- ✅ New achievements if unlocked (modal popup)
- ✅ "Continue" button (back to learn screen)
- ✅ "Review Mistakes" button (if any incorrect)

**Technical Implementation**:
- Calculate accuracy, time, XP
- Update userProgress in Firestore
- Trigger Cloud Function for achievements
- Animated XP counter (count-up effect)
- Confetti animation (react-native-confetti-cannon)

**UI Components**:
- Results card
- Stat displays
- Animated XP counter
- Confetti component
- Action buttons

**Reward Calculation**:
```typescript
const accuracy = (correctAnswers / totalAnswers) * 100
const stars = accuracy >= 95 ? 3 : accuracy >= 80 ? 2 : accuracy >= 60 ? 1 : 0
const baseXP = 50
const bonusXP = stars * 10
const totalXP = baseXP + bonusXP + (exerciseXP)
const gems = isFirstTime ? 10 : isPerfect ? 5 : 0
```

---

### 8. Hearts System (Health Points)

**Description**: Limited lives to encourage focused learning

**User Stories**:
- As a user, I want to know how many hearts I have
- As a user, I want hearts to regenerate over time
- As a user, I want to refill hearts when empty

**Acceptance Criteria**:
- ✅ Start with 5 hearts
- ✅ Lose 1 heart per wrong answer (lessons only)
- ✅ Regenerate 1 heart per 30 minutes (automatic)
- ✅ Display hearts in top-right corner (all screens)
- ✅ Show timer until next heart
- ✅ Modal when hearts reach 0 (can't start new lessons)
- ✅ Refill options: Wait, spend 50 gems, upgrade to premium
- ✅ Practice modes don't cost hearts

**Technical Implementation**:
- Firestore: `users/{id}.hearts`, `lastHeartRegenTime`
- Client-side timer for regeneration countdown
- Cloud Function to regenerate hearts (scheduled)
- Modal component for refill options

**UI Components**:
- HeartDisplay (hearts counter + timer)
- HeartRefillModal

**Regeneration Logic**:
```typescript
const HEART_REGEN_TIME = 30 * 60 * 1000 // 30 minutes
const MAX_HEARTS = 5

function getNextHeartTime(lastRegenTime: Date, currentHearts: number) {
  if (currentHearts >= MAX_HEARTS) return null
  return new Date(lastRegenTime.getTime() + HEART_REGEN_TIME)
}

function regenerateHearts(user: User) {
  const now = new Date()
  const timeSinceLastRegen = now.getTime() - user.lastHeartRegenTime.getTime()
  const heartsToAdd = Math.floor(timeSinceLastRegen / HEART_REGEN_TIME)

  if (heartsToAdd > 0) {
    const newHearts = Math.min(user.hearts + heartsToAdd, MAX_HEARTS)
    updateUser({ hearts: newHearts, lastHeartRegenTime: now })
  }
}
```

---

### 9. XP & Leveling System

**Description**: Earn XP to level up and unlock content

**User Stories**:
- As a learner, I want to earn XP for completing exercises
- As a learner, I want to level up and unlock new content
- As a learner, I want to see my progress to next level

**Acceptance Criteria**:
- ✅ Earn XP for exercises, lessons, achievements
- ✅ XP accumulates across all activities
- ✅ Progress bar shows XP to next level
- ✅ Level up triggers celebration animation
- ✅ Level badge visible on profile and home screen
- ✅ Higher levels unlock advanced lessons
- ✅ Level-up rewards: 50 gems bonus

**Technical Implementation**:
- Firestore: `users/{id}.xp`, `level`, `xpToNextLevel`
- XP calculation formulas (see GAMIFICATION.md)
- Level-up detection (client & server)
- Celebration animation on level up

**UI Components**:
- XPBar (progress bar)
- LevelBadge (icon + number)
- LevelUpModal (celebration)

**XP Sources**:
- Exercise correct: +10 XP
- Exercise perfect (first try): +15 XP
- Lesson complete: +50 XP
- Perfect lesson: +100 XP total
- Daily streak: +20 XP
- Achievements: +5 to +500 XP

---

### 10. Gems (Premium Currency)

**Description**: Premium currency for power-ups and refills

**User Stories**:
- As a user, I want to earn gems for free
- As a user, I want to spend gems on refills and hints
- As a user, I want to purchase gems if needed

**Acceptance Criteria**:
- ✅ Earn gems: first-time lessons, achievements, level-ups
- ✅ Spend gems: heart refills, hints, future power-ups
- ✅ Gem counter visible in top-right corner
- ✅ Purchase gems via in-app purchase
- ✅ Gem shop modal with packages
- ✅ Animated gem counter on earn/spend

**Technical Implementation**:
- Firestore: `users/{id}.gems`
- RevenueCat for gem purchases
- Transaction logging in Firestore
- Animated counter with sparkle effect

**UI Components**:
- GemDisplay (counter + icon)
- GemShopModal
- Purchase confirmation

**Earning Gems (Free)**:
- First lesson: +10 gems
- Perfect lesson: +5 gems
- Level up: +50 gems
- Achievements: +5-500 gems
- Daily login streak (7 days): +20 gems
- Premium monthly: +500 gems

**Spending Gems**:
- Refill hearts (5): 50 gems
- Exercise hint: 10 gems
- Streak freeze (future): 100 gems
- XP boost 2x/1hr (future): 200 gems

---

### 11. Daily Streaks

**Description**: Maintain daily learning habit

**User Stories**:
- As a learner, I want to build a daily learning habit
- As a learner, I want to see my current streak
- As a learner, I want to be reminded to maintain my streak

**Acceptance Criteria**:
- ✅ Streak increments when lesson completed (once per day)
- ✅ Streak breaks if no lesson for 24 hours
- ✅ Flame icon 🔥 with number on home screen
- ✅ Color changes based on streak length
- ✅ Push notification reminder if streak at risk
- ✅ Achievements for milestone streaks
- ✅ Longest streak tracked (all-time record)

**Technical Implementation**:
- Firestore: `users/{id}.currentStreak`, `longestStreak`, `lastLessonDate`
- Cloud Function to check streaks daily (scheduled)
- Push notification service
- Streak validation on lesson complete

**UI Components**:
- StreakFlame (icon + counter)
- Streak milestone celebration

**Streak Logic**:
```typescript
function updateStreak(user: User) {
  const now = new Date()
  const lastLesson = user.lastLessonDate

  if (!lastLesson) {
    // First lesson ever
    return { currentStreak: 1, longestStreak: 1 }
  }

  const hoursSince = (now.getTime() - lastLesson.getTime()) / (1000 * 60 * 60)

  if (hoursSince < 24) {
    // Same day, no change
    return { currentStreak: user.currentStreak }
  } else if (hoursSince < 48) {
    // Next day, increment
    const newStreak = user.currentStreak + 1
    return {
      currentStreak: newStreak,
      longestStreak: Math.max(newStreak, user.longestStreak),
    }
  } else {
    // Streak broken
    return { currentStreak: 1 }
  }
}
```

---

### 12. Achievements

**Description**: Unlock badges for milestones

**User Stories**:
- As a learner, I want to unlock achievements for my progress
- As a learner, I want to see all available achievements
- As a learner, I want to share my achievements

**Acceptance Criteria**:
- ✅ Achievements for: lessons, streaks, social, special events
- ✅ Show locked (grayscale) and unlocked (color) achievements
- ✅ Progress bar for in-progress achievements
- ✅ Popup celebration when achievement unlocked
- ✅ XP and gem rewards on unlock
- ✅ Achievement list filterable by category
- ✅ Share achievement to social media (future)

**Technical Implementation**:
- Firestore: `achievements/`, `userAchievements/{userId}_{achievementId}`
- Cloud Function checks achievements on events
- Real-time listener for achievement unlocks
- Confetti animation on unlock

**UI Components**:
- AchievementBadge
- AchievementGrid
- AchievementUnlockModal

**Achievement Examples** (see GAMIFICATION.md for full list):
- First Lesson (5 XP, 10 gems)
- 7-Day Streak (25 XP, 50 gems)
- Perfect Lesson (10 XP, 20 gems)
- Night Owl (10 XP, 20 gems)

---

### 13. Leaderboard

**Description**: Weekly competition with other learners

**User Stories**:
- As a learner, I want to compete with friends
- As a learner, I want to see my global rank
- As a learner, I want to climb the leaderboard

**Acceptance Criteria**:
- ✅ Weekly XP-based ranking
- ✅ Tabs: Friends, Global
- ✅ Show top 50 globally
- ✅ User's position pinned if not in top 50
- ✅ Shows: rank, avatar, username, weekly XP
- ✅ Position change indicator (↑↓→)
- ✅ Resets every Monday 00:00 UTC
- ✅ Add friends functionality (future)

**Technical Implementation**:
- Firestore: `leaderboards/{week}_global`
- Cloud Function updates leaderboard hourly
- Client pulls leaderboard snapshot
- Weekly reset via scheduled function

**UI Components**:
- LeaderboardEntry (list item)
- Tab selector
- User position card (pinned)

**Leaderboard Update**:
```typescript
// Hourly Cloud Function
export const updateLeaderboards = functions.pubsub
  .schedule('0 * * * *')
  .onRun(async () => {
    const week = getCurrentWeek() // "2024-W47"

    // Get top 100 users by weekly XP
    const topUsers = await getTopUsersByWeeklyXP(100)

    await firestore()
      .collection('leaderboards')
      .doc(`${week}_global`)
      .set({
        period: week,
        type: 'global',
        entries: topUsers,
        lastUpdated: new Date(),
      })
  })
```

---

### 14. Profile & Statistics

**Description**: View learning stats and progress

**User Stories**:
- As a learner, I want to see my overall stats
- As a learner, I want to see my progress over time
- As a learner, I want to see my achievements

**Acceptance Criteria**:
- ✅ Profile shows: avatar, username, level, total XP
- ✅ Stats: streak, lessons completed, accuracy, time spent
- ✅ Charts: XP per day, accuracy over time
- ✅ Time period selector: Week / Month / Year / All
- ✅ Recent activity feed
- ✅ Links to achievements, settings

**Technical Implementation**:
- Firestore: `users/{id}`, `dailyStats/{userId}_{date}`
- Charts: react-native-chart-kit or Victory Native
- Aggregate data from dailyStats

**UI Components**:
- ProfileHeader
- StatsGrid
- Charts (line, bar, area)
- ActivityFeed

**Stats Displayed**:
- Current level & XP
- Current streak & longest streak
- Total lessons completed
- Overall accuracy %
- Total time spent learning
- Words learned
- Achievements unlocked

---

### 15. Premium Subscription

**Description**: Unlock unlimited learning with subscription

**User Stories**:
- As a user, I want unlimited hearts
- As a user, I want to subscribe monthly or annually
- As a user, I want to manage my subscription

**Acceptance Criteria**:
- ✅ Two plans: Monthly ($9.99), Annual ($59.99)
- ✅ 7-day free trial (annual only)
- ✅ Premium benefits clearly listed
- ✅ Purchase via RevenueCat
- ✅ Native payment sheet (Apple/Google)
- ✅ Restore purchases button
- ✅ Manage subscription in settings
- ✅ Premium badge on profile

**Technical Implementation**:
- RevenueCat SDK integration
- Firestore: `users/{id}.isPremium`, `premiumExpiresAt`
- Webhook handling for renewals/cancellations
- Entitlement checks for premium features

**UI Components**:
- PremiumModal
- Plan selector
- Benefits list
- Purchase button

**Premium Benefits**:
- ♾️ Unlimited hearts
- 500 gems/month
- Offline lessons
- No ads (future)
- Advanced analytics
- Priority support

---

## Future Features (Post-MVP)

### 16. Speaking Practice

**Description**: Pronunciation practice with speech recognition

**Features**:
- Record voice
- Compare with native speaker
- Pronunciation scoring
- Feedback on specific sounds

---

### 17. Conversation Practice

**Description**: AI-powered conversation partner

**Features**:
- Chat with AI in target language
- Real-time translation
- Contextual corrections
- Difficulty levels

---

### 18. Stories

**Description**: Read stories in target language

**Features**:
- Graded readers (beginner to advanced)
- Tap words for instant translation
- Audio narration
- Comprehension quizzes

---

### 19. Grammar Lessons

**Description**: Dedicated grammar explanations

**Features**:
- Grammar rules explained
- Examples with translations
- Practice exercises
- Reference guide

---

### 20. Social Features

**Description**: Learn with friends

**Features**:
- Add friends
- Challenge friends
- Group learning
- Share progress
- Friend chat

---

### 21. Offline Mode

**Description**: Download lessons for offline use

**Features**:
- Download lessons
- Offline progress tracking
- Auto-sync when online
- Storage management

---

### 22. Multiple Languages

**Description**: Learn additional languages

**Features**:
- Spanish, German, Portuguese, French, Ukrainian
- Switch learning language
- Track progress separately
- Polyglot achievements

---

This feature specification covers all MVP features in detail. Each feature should be implemented following the technical specifications and tested thoroughly before release.
