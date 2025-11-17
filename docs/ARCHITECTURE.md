# Technical Architecture

## System Overview

Lingsy is a mobile-first React Native application built with Expo, using Firebase as the backend infrastructure and RevenueCat for payment management.

```
┌─────────────────────────────────────────────────────────────┐
│                         Mobile App                          │
│           (React Native + Expo + TypeScript)               │
│                                                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │  Screens │  │Components│  │ Services │  │  Models  │  │
│  │  (Views) │  │   (UI)   │  │  (API)   │  │  (MST)   │  │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘  │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │              Expo Router (Navigation)                 │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                           ↕ HTTP/WebSocket
┌─────────────────────────────────────────────────────────────┐
│                     Firebase Backend                        │
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │  Firestore   │  │     Auth     │  │   Storage    │    │
│  │  (Database)  │  │ (Users/Auth) │  │ (Media Files)│    │
│  └──────────────┘  └──────────────┘  └──────────────┘    │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │        Cloud Functions (Serverless Backend)          │  │
│  │  • Achievement checks  • Leaderboard updates        │  │
│  │  • Payment webhooks    • Data aggregation           │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                           ↕ API Calls
┌─────────────────────────────────────────────────────────────┐
│                    External Services                        │
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │  RevenueCat  │  │    Stripe    │  │   Analytics  │    │
│  │(Subscriptions)│ │  (Payments)  │  │   (Mixpanel) │    │
│  └──────────────┘  └──────────────┘  └──────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

---

## Mobile Application Architecture

### Tech Stack

**Core**:
- **React Native**: 0.81.5
- **Expo**: SDK 54
- **TypeScript**: 5.9.2
- **Node**: >=20.0.0

**Navigation**:
- **Expo Router**: 6.0 (file-based routing)
- **React Navigation**: 7.0 (underlying)

**State Management**:
- **MobX-State-Tree**: Ignite default, reactive state
- **React Context**: For theme, i18n
- **MMKV**: Fast local storage

**UI/UX**:
- **React Native Reanimated**: 4.1 (animations)
- **React Native Gesture Handler**: 2.28 (gestures)
- **Custom components**: Theme-based design system

**Internationalization**:
- **i18next**: 23.14
- **react-i18next**: 15.0
- **expo-localization**: Device locale detection

**Backend Integration**:
- **Firebase SDK**:
  - `@react-native-firebase/app`
  - `@react-native-firebase/auth`
  - `@react-native-firebase/firestore`
  - `@react-native-firebase/storage`
  - `@react-native-firebase/analytics`
  - `@react-native-firebase/messaging` (push notifications)

**Payments**:
- **react-native-purchases**: RevenueCat SDK
- **@stripe/stripe-react-native**: Stripe SDK (future web payments)

**Media**:
- **expo-av**: Audio playback and recording
- **expo-image**: Optimized image loading
- **react-native-fast-image**: Image caching

**Development**:
- **Reactotron**: Debugging
- **Jest**: Unit testing
- **React Testing Library**: Component testing
- **Maestro**: E2E testing
- **ESLint + Prettier**: Code quality

---

## Application Structure

```
src/
├── app/                          # Expo Router screens (file-based routing)
│   ├── (auth)/                  # Auth stack (no tabs)
│   │   ├── _layout.tsx          # Auth stack layout
│   │   ├── welcome.tsx
│   │   ├── login.tsx
│   │   ├── signup.tsx
│   │   └── language-select.tsx
│   │
│   ├── (tabs)/                  # Main app (bottom tabs)
│   │   ├── _layout.tsx          # Tab layout
│   │   ├── learn/
│   │   │   ├── _layout.tsx
│   │   │   ├── index.tsx        # Learn home
│   │   │   ├── lesson/[id].tsx  # Lesson detail
│   │   │   └── exercise/[type]/[id].tsx
│   │   ├── practice/
│   │   │   ├── index.tsx
│   │   │   └── flashcards.tsx
│   │   ├── leaderboard/
│   │   │   └── index.tsx
│   │   └── profile/
│   │       ├── index.tsx
│   │       ├── settings.tsx
│   │       ├── statistics.tsx
│   │       └── achievements.tsx
│   │
│   ├── (modals)/               # Modal screens
│   │   ├── shop.tsx            # Gem shop
│   │   ├── premium.tsx         # Subscription
│   │   ├── health.tsx          # Heart refill
│   │   └── achievement-unlock.tsx
│   │
│   └── _layout.tsx             # Root layout
│
├── components/                  # Reusable components
│   ├── common/                 # Basic UI components
│   │   ├── Button.tsx
│   │   ├── Text.tsx
│   │   ├── Card.tsx
│   │   ├── Icon.tsx
│   │   ├── TextField.tsx
│   │   ├── ProgressBar.tsx
│   │   └── Avatar.tsx
│   │
│   ├── learning/               # Learning-specific components
│   │   ├── LessonCard.tsx
│   │   ├── ExerciseContainer.tsx
│   │   ├── FlashCard.tsx
│   │   ├── MultipleChoice.tsx
│   │   ├── WritingInput.tsx
│   │   └── AudioPlayer.tsx
│   │
│   ├── gamification/           # Game elements
│   │   ├── XPBar.tsx
│   │   ├── LevelBadge.tsx
│   │   ├── HeartDisplay.tsx
│   │   ├── StreakFlame.tsx
│   │   ├── AchievementBadge.tsx
│   │   └── Confetti.tsx
│   │
│   └── layout/                 # Layout components
│       ├── Screen.tsx
│       ├── Header.tsx
│       └── TabBar.tsx
│
├── models/                      # MobX-State-Tree models
│   ├── RootStore.ts            # Root store
│   ├── UserStore.ts            # User state & auth
│   ├── LessonStore.ts          # Lessons data
│   ├── ProgressStore.ts        # Learning progress
│   ├── GameStore.ts            # Gamification state
│   ├── SubscriptionStore.ts    # Premium status
│   └── helpers/                # Store helpers
│       ├── withRootStore.ts
│       └── setupRootStore.ts
│
├── services/                    # External service integrations
│   ├── api/                    # API client
│   │   ├── api.ts              # Apisauce client
│   │   ├── api.types.ts        # API types
│   │   └── api-config.ts       # Config
│   │
│   ├── firebase/               # Firebase services
│   │   ├── firebaseConfig.ts   # Firebase init
│   │   ├── auth.ts             # Authentication
│   │   ├── firestore.ts        # Database operations
│   │   ├── storage.ts          # File storage
│   │   └── analytics.ts        # Analytics events
│   │
│   ├── audio/                  # Audio service
│   │   ├── AudioService.ts     # Audio playback
│   │   └── AudioCache.ts       # Audio caching
│   │
│   ├── subscription/           # Payment service
│   │   └── RevenueCatService.ts
│   │
│   └── push/                   # Push notifications
│       └── PushService.ts
│
├── utils/                       # Utility functions
│   ├── formatters/             # Data formatting
│   │   ├── date.ts
│   │   └── number.ts
│   ├── validators/             # Input validation
│   │   └── forms.ts
│   ├── storage/                # Async storage
│   │   └── mmkv.ts
│   └── helpers/                # Helper functions
│       ├── xp.ts               # XP calculations
│       ├── hearts.ts           # Heart logic
│       └── streaks.ts          # Streak validation
│
├── theme/                       # Design system
│   ├── index.ts                # Theme exports
│   ├── colors.ts               # Color palette
│   ├── spacing.ts              # Spacing scale
│   ├── typography.ts           # Font styles
│   └── animations.ts           # Animation configs
│
├── i18n/                        # Translations
│   ├── index.ts
│   ├── en.ts                   # English
│   └── uk.ts                   # Ukrainian
│
├── config/                      # App configuration
│   └── config.ts               # Environment config
│
└── types/                       # TypeScript types
    ├── models.ts               # Data models
    ├── navigation.ts           # Navigation types
    └── firebase.ts             # Firebase types
```

---

## State Management (MobX-State-Tree)

### RootStore

```typescript
export const RootStoreModel = types.model("RootStore").props({
  userStore: types.optional(UserStoreModel, {}),
  lessonStore: types.optional(LessonStoreModel, {}),
  progressStore: types.optional(ProgressStoreModel, {}),
  gameStore: types.optional(GameStoreModel, {}),
  subscriptionStore: types.optional(SubscriptionStoreModel, {}),
})
```

### UserStore

```typescript
export const UserStoreModel = types
  .model("UserStore")
  .props({
    currentUser: types.maybeNull(UserModel),
    isAuthenticated: types.optional(types.boolean, false),
    authToken: types.maybeNull(types.string),
  })
  .actions((self) => ({
    setUser(user: User) {
      self.currentUser = user
      self.isAuthenticated = true
    },
    async login(email: string, password: string) {
      // Firebase auth
      const userCredential = await auth().signInWithEmailAndPassword(email, password)
      const userData = await firestore().collection('users').doc(userCredential.user.uid).get()
      this.setUser(userData.data())
    },
    async logout() {
      await auth().signOut()
      self.currentUser = null
      self.isAuthenticated = false
    },
  }))
  .views((self) => ({
    get isPremium() {
      return self.currentUser?.isPremium ?? false
    },
    get hearts() {
      return self.currentUser?.hearts ?? 5
    },
    get gems() {
      return self.currentUser?.gems ?? 0
    },
  }))
```

### GameStore

```typescript
export const GameStoreModel = types
  .model("GameStore")
  .props({
    currentStreak: types.optional(types.number, 0),
    longestStreak: types.optional(types.number, 0),
    xp: types.optional(types.number, 0),
    level: types.optional(types.number, 1),
    comboMultiplier: types.optional(types.number, 1),
  })
  .actions((self) => ({
    addXP(amount: number) {
      const totalXP = amount * self.comboMultiplier
      self.xp += totalXP
      this.checkLevelUp()
    },
    checkLevelUp() {
      const requiredXP = calculateXPForLevel(self.level + 1)
      if (self.xp >= requiredXP) {
        self.level += 1
        // Trigger celebration
      }
    },
    incrementStreak() {
      self.currentStreak += 1
      if (self.currentStreak > self.longestStreak) {
        self.longestStreak = self.currentStreak
      }
    },
  }))
```

---

## Data Flow

### Authentication Flow

```
User enters email/password
  → UserStore.login()
    → Firebase Auth.signInWithEmailAndPassword()
      → Success: Get user data from Firestore
        → UserStore.setUser()
          → Navigate to main app
      → Error: Show error message
```

### Lesson Completion Flow

```
User completes lesson
  → Calculate results (accuracy, XP, etc.)
    → Update UserProgress in Firestore
    → Update local GameStore (XP, streak, etc.)
    → Check achievements (Cloud Function)
      → If achievement unlocked:
        → Show achievement modal
    → Update daily stats
    → Navigate to lesson complete screen
```

### Payment Flow

```
User taps "Buy Gems"
  → SubscriptionStore.purchaseGems(packageId)
    → RevenueCat.purchasePackage()
      → Native payment sheet
        → User confirms
          → Payment processed
            → RevenueCat webhook → Cloud Function
              → Update Firestore (user.gems)
                → Listen to Firestore change
                  → Update local UserStore
                    → Show success message
```

---

## Firebase Backend Architecture

### Firestore Collections

See `DATABASE_SCHEMA.md` for full schema.

**Key Collections**:
- `users/` - User profiles and stats
- `lessons/` - Lesson content
- `exercises/` - Exercise definitions
- `vocabularyItems/` - Vocabulary database
- `userProgress/` - Learning progress per user/lesson
- `userAchievements/` - Achievement progress
- `leaderboards/` - Leaderboard snapshots

### Cloud Functions

**Triggers**:

1. **onUserCreate** (Auth trigger)
   ```typescript
   // When user signs up
   export const onUserCreate = functions.auth.user().onCreate(async (user) => {
     await firestore().collection('users').doc(user.uid).set({
       email: user.email,
       level: 1,
       xp: 0,
       hearts: 5,
       gems: 0,
       // ... defaults
     })

     // Create achievement progress docs
     await initializeUserAchievements(user.uid)
   })
   ```

2. **onLessonComplete** (Firestore trigger)
   ```typescript
   // When userProgress/{id} is created/updated with isCompleted=true
   export const onLessonComplete = functions.firestore
     .document('userProgress/{progressId}')
     .onWrite(async (change, context) => {
       const newData = change.after.data()

       if (newData.isCompleted) {
         const userId = newData.userId

         // Update user stats
         await updateUserStats(userId, newData)

         // Check achievements
         await checkAchievements(userId)

         // Update leaderboard
         await updateLeaderboard(userId, newData.xpEarned)

         // Check streak
         await checkAndUpdateStreak(userId)
       }
     })
   ```

3. **updateLeaderboards** (Scheduled function)
   ```typescript
   // Every hour, update leaderboard snapshots
   export const updateLeaderboards = functions.pubsub
     .schedule('0 * * * *') // Every hour
     .onRun(async (context) => {
       await generateGlobalLeaderboard()
       await generateCountryLeaderboards()
     })
   ```

4. **revenuecatWebhook** (HTTP function)
   ```typescript
   // Handle payment webhooks from RevenueCat
   export const revenuecatWebhook = functions.https.onRequest(async (req, res) => {
     const event = req.body.event

     switch (event.type) {
       case 'INITIAL_PURCHASE':
       case 'RENEWAL':
         await grantPremiumAccess(event.app_user_id)
         break
       case 'CANCELLATION':
         await revokePremiumAccess(event.app_user_id)
         break
       case 'NON_RENEWING_PURCHASE':
         await grantGems(event.app_user_id, event.product_id)
         break
     }

     res.sendStatus(200)
   })
   ```

5. **generateDailyStats** (Scheduled function)
   ```typescript
   // Every day at midnight UTC, aggregate stats
   export const generateDailyStats = functions.pubsub
     .schedule('0 0 * * *')
     .onRun(async (context) => {
       await aggregateDailyStats()
     })
   ```

---

## Offline Support

### Strategy: Offline-First Architecture

**Local Storage (MMKV)**:
- User profile
- Current lesson progress
- Completed exercises
- Downloaded audio files
- Cached images

**Firestore Offline Persistence**:
```typescript
// Enable offline persistence
await firestore().settings({
  persistence: true,
  cacheSizeBytes: firestore.CACHE_SIZE_UNLIMITED,
})
```

**Data Sync**:
1. User completes lesson offline
2. Save to local MMKV + Firestore (queued)
3. Firestore SDK syncs when online
4. Cloud Functions process on sync

**Audio Caching**:
```typescript
// Download audio files for offline use
export class AudioCache {
  async downloadLesson(lessonId: string) {
    const exercises = await getExercises(lessonId)
    for (const exercise of exercises) {
      if (exercise.audioUrl) {
        await FileSystem.downloadAsync(
          exercise.audioUrl,
          this.getCachePath(exercise.id)
        )
      }
    }
  }
}
```

**Premium Feature**:
- Free users: Current lesson only
- Premium users: Download multiple lessons

---

## Performance Optimization

### Bundle Size
- Code splitting with dynamic imports
- Remove unused dependencies
- Tree shaking enabled
- Hermes JS engine (faster startup)

### Image Optimization
- Use `expo-image` with caching
- Lazy load images
- WebP format
- Multiple resolutions (1x, 2x, 3x)

### List Performance
- FlatList with `getItemLayout`
- `windowSize` optimization
- `removeClippedSubviews={true}`
- Memoized list items

### Animation Performance
- Use Reanimated (runs on UI thread)
- Avoid layout animations
- Use `useNativeDriver` when possible
- Limit concurrent animations

### Network Optimization
- GraphQL batching (if using GraphQL)
- Request debouncing
- Pagination (infinite scroll)
- Prefetch next lesson

---

## Security

### Authentication
- Firebase Auth with email/password
- Email verification required
- Password reset flow
- Session management

### Authorization
- Firestore security rules (see DATABASE_SCHEMA.md)
- Cloud Functions validate all writes
- User can only modify own data
- Content is read-only (admin writes via Cloud Functions)

### Data Protection
- HTTPS only
- Encrypted storage (MMKV)
- No sensitive data in logs
- Secure environment variables

### API Keys
- Environment variables (not in repo)
- Different keys per environment (dev/staging/prod)
- Rotate keys regularly
- Restrict API key usage (domain/bundle ID)

---

## Monitoring & Analytics

### Firebase Analytics
```typescript
import analytics from '@react-native-firebase/analytics'

// Track events
await analytics().logEvent('lesson_completed', {
  lesson_id: lessonId,
  accuracy: 95,
  time_spent: 180,
})
```

**Key Events**:
- `app_open`
- `sign_up`
- `login`
- `lesson_started`
- `lesson_completed`
- `exercise_answered`
- `achievement_unlocked`
- `purchase_initiated`
- `purchase_completed`
- `subscription_started`

### Crashlytics
```typescript
import crashlytics from '@react-native-firebase/crashlytics'

// Log errors
crashlytics().recordError(error)

// Set user context
crashlytics().setUserId(userId)
```

### Performance Monitoring
```typescript
import perf from '@react-native-firebase/perf'

// Trace lesson load time
const trace = await perf().startTrace('lesson_load')
await loadLesson(id)
await trace.stop()
```

### Custom Metrics
- DAU/MAU/WAU
- Retention rates (D1, D7, D30)
- Session length
- Lessons per session
- Accuracy rates
- Churn prediction
- LTV estimation

---

## Testing Strategy

### Unit Tests (Jest)
```typescript
// models/GameStore.test.ts
describe('GameStore', () => {
  it('should add XP correctly', () => {
    const store = GameStoreModel.create({ xp: 0 })
    store.addXP(100)
    expect(store.xp).toBe(100)
  })

  it('should level up at threshold', () => {
    const store = GameStoreModel.create({ xp: 0, level: 1 })
    store.addXP(100)
    expect(store.level).toBe(2)
  })
})
```

### Component Tests (React Testing Library)
```typescript
// components/LessonCard.test.tsx
describe('LessonCard', () => {
  it('should render lesson title', () => {
    const { getByText } = render(<LessonCard lesson={mockLesson} />)
    expect(getByText('Lesson 1')).toBeTruthy()
  })

  it('should show lock icon if locked', () => {
    const { getByTestId } = render(
      <LessonCard lesson={{ ...mockLesson, isLocked: true }} />
    )
    expect(getByTestId('lock-icon')).toBeTruthy()
  })
})
```

### E2E Tests (Maestro)
```yaml
# .maestro/flows/complete-lesson.yaml
appId: com.lingsy
---
- launchApp
- tapOn: "Learn"
- tapOn: "Lesson 1"
- tapOn: "Start Lesson"
- repeat:
    times: 5
    commands:
      - tapOn: "Option A"
      - tapOn: "Continue"
- assertVisible: "Lesson Complete!"
```

### Integration Tests
- Firebase emulator for backend testing
- Mock RevenueCat purchases
- Test offline sync
- Test Cloud Functions locally

---

## CI/CD Pipeline

### GitHub Actions

```yaml
name: CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm install
      - run: npm run lint
      - run: npm run compile
      - run: npm test

  build:
    needs: test
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v3
      - uses: expo/expo-github-action@v8
      - run: npm install
      - run: eas build --platform ios --profile preview
      - run: eas build --platform android --profile preview

  deploy:
    needs: build
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - run: eas submit --platform ios
      - run: eas submit --platform android
```

---

## Deployment

### Environments

**Development**:
- Firebase: `lingsy-dev`
- RevenueCat: Sandbox
- Stripe: Test mode

**Staging**:
- Firebase: `lingsy-staging`
- RevenueCat: Sandbox
- Stripe: Test mode
- TestFlight/Internal Testing

**Production**:
- Firebase: `lingsy-prod`
- RevenueCat: Production
- Stripe: Live mode
- App Store/Play Store

### Release Process

1. **Development** → `develop` branch
2. **Feature complete** → Create release branch
3. **QA testing** → Fix bugs
4. **Staging build** → TestFlight/Internal Testing
5. **User testing** → Gather feedback
6. **Production build** → Submit to stores
7. **Phased rollout** → 10% → 50% → 100%
8. **Monitor** → Crashlytics, Analytics
9. **Hotfix if needed** → Fast-track critical fixes

---

This architecture is designed for scalability, maintainability, and performance. All components should be tested thoroughly before production deployment.
