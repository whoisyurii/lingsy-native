# Firestore Database Schema

## Collections Overview

```
users/                          # User profiles
lessons/                        # Lesson content
exercises/                      # Exercise content
vocabularyItems/                # Vocabulary database
achievements/                   # Achievement definitions
subscriptions/                  # User subscriptions (managed by RevenueCat)
userProgress/                   # User learning progress
userAchievements/               # User achievement progress
leaderboards/                   # Leaderboard data
```

---

## 1. users/ Collection

**Document ID**: Firebase Auth UID

```typescript
interface User {
  // Basic Info
  uid: string;
  email: string;
  username: string;
  displayName: string;
  avatarUrl?: string;

  // Language Settings
  interfaceLanguage: 'en' | 'uk';
  learningLanguage: 'en'; // MVP: only English

  // Gamification
  level: number;                    // Current level
  xp: number;                       // Total XP earned
  xpToNextLevel: number;            // XP needed for next level
  gems: number;                     // Premium currency
  hearts: number;                   // Health points (max 5)
  lastHeartRegenTime: Timestamp;    // When last heart regenerated

  // Streaks
  currentStreak: number;            // Days
  longestStreak: number;            // Days
  lastLessonDate: Timestamp;        // Date of last lesson completion

  // Stats
  totalLessonsCompleted: number;
  totalExercisesCompleted: number;
  accuracyRate: number;             // Percentage
  totalTimeSpentMinutes: number;
  wordsLearned: number;

  // Premium
  isPremium: boolean;
  premiumExpiresAt?: Timestamp;
  revenueCatUserId?: string;

  // Settings
  settings: {
    notifications: {
      dailyReminder: boolean;
      dailyReminderTime: string;    // "09:00"
      streakReminder: boolean;
      achievementAlerts: boolean;
    };
    audio: {
      soundEffects: boolean;
      autoplay: boolean;
      volume: number;               // 0-1
    };
    haptics: boolean;
    offlineMode: boolean;
  };

  // Metadata
  createdAt: Timestamp;
  updatedAt: Timestamp;
  lastSeenAt: Timestamp;
  deviceTokens: string[];           // For push notifications
}
```

**Indexes**:
- `xp` (descending) - for leaderboards
- `currentStreak` (descending) - for streak leaderboards
- `createdAt` (ascending) - for cohort analysis

---

## 2. lessons/ Collection

**Document ID**: Auto-generated

```typescript
interface Lesson {
  id: string;

  // Metadata
  title: {
    en: string;
    uk: string;
  };
  description: {
    en: string;
    uk: string;
  };

  // Ordering
  order: number;                    // Sequential order in curriculum
  level: number;                    // Difficulty level (1-5)

  // Content
  learningLanguage: 'en';           // Target language
  exerciseIds: string[];            // Ordered list of exercise IDs
  vocabularyIds: string[];          // Words introduced in this lesson

  // Requirements
  prerequisiteLessonIds: string[];  // Must complete these first
  requiredLevel: number;            // User level requirement

  // Rewards
  xpReward: number;                 // Base XP
  gemsReward: number;               // First-time completion

  // Difficulty
  estimatedMinutes: number;         // 5-10 typically
  difficulty: 'beginner' | 'intermediate' | 'advanced';

  // Metadata
  isPublished: boolean;
  createdAt: Timestamp;
  updatedAt: Timestamp;
  createdBy: string;                // Admin user ID
}
```

**Indexes**:
- `order` (ascending) - for lesson path
- `isPublished, order` (composite) - for published lessons only
- `learningLanguage, level` (composite) - for filtering

---

## 3. exercises/ Collection

**Document ID**: Auto-generated

```typescript
type ExerciseType = 'flashcard' | 'listening' | 'writing' | 'multipleChoice';

interface BaseExercise {
  id: string;
  type: ExerciseType;
  lessonId: string;
  order: number;                    // Order within lesson

  // Content
  question: {
    en: string;
    uk: string;
  };

  // Media
  audioUrl?: string;                // Cloud Storage URL
  imageUrl?: string;                // Cloud Storage URL

  // Difficulty
  difficulty: 1 | 2 | 3;
  xpReward: number;                 // 10 typically

  // Metadata
  createdAt: Timestamp;
  updatedAt: Timestamp;
}

// Flashcard Exercise
interface FlashcardExercise extends BaseExercise {
  type: 'flashcard';
  front: {
    text: { en: string; uk: string; };
    imageUrl?: string;
    audioUrl?: string;
  };
  back: {
    text: { en: string; uk: string; };
    imageUrl?: string;
  };
}

// Listening Exercise
interface ListeningExercise extends BaseExercise {
  type: 'listening';
  audioUrl: string;                 // Required
  correctAnswer: string;            // English text
  options: string[];                // 4 options
  hint?: { en: string; uk: string; };
}

// Writing Exercise
interface WritingExercise extends BaseExercise {
  type: 'writing';
  audioUrl: string;                 // Play the word/phrase
  correctAnswer: string;            // Exact match required
  acceptedAnswers?: string[];       // Alternative correct answers
  hint?: string;                    // First letter, etc.
}

// Multiple Choice Exercise (future)
interface MultipleChoiceExercise extends BaseExercise {
  type: 'multipleChoice';
  question: { en: string; uk: string; };
  options: string[];
  correctOptionIndex: number;
}

type Exercise = FlashcardExercise | ListeningExercise | WritingExercise | MultipleChoiceExercise;
```

**Indexes**:
- `lessonId, order` (composite) - for lesson exercises
- `type` - for practice modes

---

## 4. vocabularyItems/ Collection

**Document ID**: Auto-generated

```typescript
interface VocabularyItem {
  id: string;

  // Word/Phrase
  word: string;                     // In learning language (English)
  translation: {
    uk: string;                     // Ukrainian translation
  };

  // Grammar
  partOfSpeech: 'noun' | 'verb' | 'adjective' | 'adverb' | 'phrase' | 'other';
  gender?: 'masculine' | 'feminine' | 'neuter';  // For nouns in gendered languages

  // Context
  exampleSentence?: {
    en: string;
    uk: string;
  };

  // Media
  imageUrl?: string;
  audioUrl: string;                 // Pronunciation

  // Categorization
  topics: string[];                 // ['greetings', 'food', 'travel']
  difficulty: 1 | 2 | 3 | 4 | 5;
  cefr?: 'A1' | 'A2' | 'B1' | 'B2' | 'C1' | 'C2';

  // Usage
  frequency: number;                // How common (1-10)

  // Metadata
  createdAt: Timestamp;
  updatedAt: Timestamp;
}
```

**Indexes**:
- `topics` (array-contains) - for topic-based practice
- `difficulty` - for filtering
- `frequency` (descending) - for prioritization

---

## 5. achievements/ Collection

**Document ID**: Unique slug (e.g., "first-lesson")

```typescript
interface Achievement {
  id: string;                       // Slug

  // Display
  title: {
    en: string;
    uk: string;
  };
  description: {
    en: string;
    uk: string;
  };
  iconUrl: string;
  iconUrlLocked: string;            // Grayscale version

  // Category
  category: 'learning' | 'streaks' | 'social' | 'special';

  // Requirements
  requirement: {
    type: 'lessons_completed' | 'streak_days' | 'perfect_lessons' | 'xp_earned' | 'custom';
    value: number;                  // Target value
    customValidator?: string;       // Cloud Function name for complex logic
  };

  // Rewards
  xpReward: number;
  gemsReward: number;

  // Display
  rarity: 'common' | 'rare' | 'epic' | 'legendary';
  order: number;                    // Display order
  isSecret: boolean;                // Hide until unlocked

  // Metadata
  createdAt: Timestamp;
  updatedAt: Timestamp;
}
```

**Indexes**:
- `category, order` (composite) - for displaying by category

---

## 6. userProgress/ Collection

**Document ID**: `{userId}_{lessonId}`

```typescript
interface UserProgress {
  id: string;
  userId: string;
  lessonId: string;

  // Completion
  status: 'not_started' | 'in_progress' | 'completed';
  isCompleted: boolean;
  completedAt?: Timestamp;

  // Performance
  attempts: number;                 // How many times started
  correctAnswers: number;
  totalAnswers: number;
  accuracy: number;                 // Percentage
  timeSpentSeconds: number;

  // Rewards
  xpEarned: number;
  gemsEarned: number;

  // Stars (0-3 based on accuracy)
  stars: 0 | 1 | 2 | 3;

  // Exercises
  exerciseProgress: {
    [exerciseId: string]: {
      isCorrect: boolean;
      attempts: number;
      completedAt: Timestamp;
    };
  };

  // Metadata
  firstAttemptAt: Timestamp;
  lastAttemptAt: Timestamp;
  updatedAt: Timestamp;
}
```

**Indexes**:
- `userId, lastAttemptAt` (composite) - for recent activity
- `userId, isCompleted` (composite) - for completed lessons
- `lessonId, accuracy` (composite) - for lesson analytics

---

## 7. userVocabulary/ Collection

**Document ID**: `{userId}_{vocabularyItemId}`

```typescript
interface UserVocabulary {
  id: string;
  userId: string;
  vocabularyItemId: string;

  // Learning Status
  masteryLevel: 0 | 1 | 2 | 3 | 4 | 5;  // 0 = new, 5 = mastered

  // Spaced Repetition (Leitner System)
  box: 1 | 2 | 3 | 4 | 5;           // Which "box" in Leitner system
  nextReviewDate: Timestamp;        // When to review next

  // Performance
  timesReviewed: number;
  timesCorrect: number;
  timesIncorrect: number;
  accuracy: number;                 // Percentage

  // Tracking
  isFavorite: boolean;
  isMarkedForReview: boolean;

  // Dates
  firstSeenAt: Timestamp;
  lastReviewedAt: Timestamp;
  masteredAt?: Timestamp;           // When reached mastery level 5

  updatedAt: Timestamp;
}
```

**Indexes**:
- `userId, nextReviewDate` (composite) - for review queue
- `userId, isFavorite` (composite) - for favorites
- `userId, masteryLevel` (composite) - for mastery tracking

---

## 8. userAchievements/ Collection

**Document ID**: `{userId}_{achievementId}`

```typescript
interface UserAchievement {
  id: string;
  userId: string;
  achievementId: string;

  // Status
  isUnlocked: boolean;
  progress: number;                 // Current progress
  target: number;                   // Target value
  percentage: number;               // progress / target * 100

  // Rewards
  xpEarned: number;
  gemsEarned: number;

  // Dates
  unlockedAt?: Timestamp;
  createdAt: Timestamp;
  updatedAt: Timestamp;
}
```

**Indexes**:
- `userId, isUnlocked` (composite) - for user achievements
- `userId, percentage` (composite) - for sorting by progress

---

## 9. leaderboards/ Collection

**Document ID**: `{period}_{type}` (e.g., "2024-W47_global")

```typescript
interface Leaderboard {
  id: string;
  type: 'global' | 'country' | 'friends';
  period: string;                   // "2024-W47" for weekly, "2024-11" for monthly

  // Timeframe
  startDate: Timestamp;
  endDate: Timestamp;

  // Entries
  entries: LeaderboardEntry[];      // Top 100 only

  // Metadata
  lastUpdated: Timestamp;
  isActive: boolean;                // Current period
}

interface LeaderboardEntry {
  userId: string;
  username: string;
  avatarUrl?: string;
  xp: number;                       // XP earned this period
  rank: number;
  country?: string;                 // ISO country code
  previousRank?: number;            // For showing ↑↓
}
```

**Indexes**:
- `type, period, isActive` (composite) - for current leaderboards

---

## 10. transactions/ Collection

**Document ID**: Auto-generated

```typescript
interface Transaction {
  id: string;
  userId: string;

  // Type
  type: 'gems_purchase' | 'gems_spent' | 'hearts_refill' | 'subscription';

  // Details
  amount: number;                   // Gems or currency
  currency?: string;                // USD, EUR, etc.

  // Item
  itemType: 'gems' | 'hearts' | 'premium' | 'hint';
  itemId?: string;

  // Payment
  paymentProvider?: 'stripe' | 'apple' | 'google';
  paymentId?: string;               // External transaction ID

  // Status
  status: 'pending' | 'completed' | 'failed' | 'refunded';

  // Metadata
  createdAt: Timestamp;
  completedAt?: Timestamp;
}
```

**Indexes**:
- `userId, createdAt` (composite) - for transaction history
- `status` - for pending transactions

---

## 11. dailyStats/ Collection

**Document ID**: `{userId}_{date}` (e.g., "user123_2024-11-17")

```typescript
interface DailyStats {
  id: string;
  userId: string;
  date: string;                     // YYYY-MM-DD

  // Activity
  lessonsCompleted: number;
  exercisesCompleted: number;
  timeSpentMinutes: number;

  // Performance
  correctAnswers: number;
  totalAnswers: number;
  accuracy: number;

  // Rewards
  xpEarned: number;
  gemsEarned: number;

  // Streaks
  maintainedStreak: boolean;

  // Metadata
  createdAt: Timestamp;
  updatedAt: Timestamp;
}
```

**Indexes**:
- `userId, date` (composite) - for user stats
- `date` - for aggregation

---

## Security Rules

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Helper functions
    function isAuthenticated() {
      return request.auth != null;
    }

    function isOwner(userId) {
      return request.auth.uid == userId;
    }

    function isPremium() {
      return get(/databases/$(database)/documents/users/$(request.auth.uid)).data.isPremium == true;
    }

    // Users - users can read and write their own data
    match /users/{userId} {
      allow read: if isAuthenticated();
      allow write: if isOwner(userId);
    }

    // Lessons - read-only for all authenticated users
    match /lessons/{lessonId} {
      allow read: if isAuthenticated();
      allow write: if false;  // Only admin via Cloud Functions
    }

    // Exercises - read-only for all authenticated users
    match /exercises/{exerciseId} {
      allow read: if isAuthenticated();
      allow write: if false;  // Only admin via Cloud Functions
    }

    // Vocabulary - read-only for all authenticated users
    match /vocabularyItems/{itemId} {
      allow read: if isAuthenticated();
      allow write: if false;  // Only admin via Cloud Functions
    }

    // Achievements - read-only for all authenticated users
    match /achievements/{achievementId} {
      allow read: if isAuthenticated();
      allow write: if false;  // Only admin via Cloud Functions
    }

    // User Progress - users can read and write their own
    match /userProgress/{progressId} {
      allow read: if isAuthenticated() &&
                     progressId.matches('^' + request.auth.uid + '_.*');
      allow write: if isAuthenticated() &&
                      progressId.matches('^' + request.auth.uid + '_.*');
    }

    // User Vocabulary - users can read and write their own
    match /userVocabulary/{vocabId} {
      allow read: if isAuthenticated() &&
                     vocabId.matches('^' + request.auth.uid + '_.*');
      allow write: if isAuthenticated() &&
                      vocabId.matches('^' + request.auth.uid + '_.*');
    }

    // User Achievements - users can read their own, Cloud Functions write
    match /userAchievements/{achievementId} {
      allow read: if isAuthenticated() &&
                     achievementId.matches('^' + request.auth.uid + '_.*');
      allow write: if false;  // Only Cloud Functions
    }

    // Leaderboards - read-only for all authenticated users
    match /leaderboards/{leaderboardId} {
      allow read: if isAuthenticated();
      allow write: if false;  // Only Cloud Functions
    }

    // Transactions - users can read their own
    match /transactions/{transactionId} {
      allow read: if isAuthenticated() &&
                     resource.data.userId == request.auth.uid;
      allow write: if false;  // Only Cloud Functions
    }

    // Daily Stats - users can read and write their own
    match /dailyStats/{statId} {
      allow read: if isAuthenticated() &&
                     statId.matches('^' + request.auth.uid + '_.*');
      allow write: if isAuthenticated() &&
                      statId.matches('^' + request.auth.uid + '_.*');
    }
  }
}
```

---

## Cloud Storage Structure

```
/audio/
  /vocabulary/
    /{vocabularyItemId}.mp3
  /exercises/
    /{exerciseId}.mp3

/images/
  /vocabulary/
    /{vocabularyItemId}.jpg
  /exercises/
    /{exerciseId}.jpg
  /achievements/
    /{achievementId}.png
    /{achievementId}_locked.png

/avatars/
  /{userId}.jpg
```

---

## Data Initialization

On user signup, Cloud Function creates:

```typescript
// users/{uid}
{
  level: 1,
  xp: 0,
  xpToNextLevel: 100,
  gems: 0,
  hearts: 5,
  currentStreak: 0,
  longestStreak: 0,
  totalLessonsCompleted: 0,
  accuracyRate: 0,
  isPremium: false,
  // ... other defaults
}

// userAchievements/{uid}_{achievementId} for all achievements
{
  isUnlocked: false,
  progress: 0,
  target: achievement.requirement.value,
  percentage: 0,
}
```

---

## Backup & Data Export

- Automated daily Firestore backups
- User data export API (GDPR compliance)
- Data retention: 7 years for financial, 3 years for user data
- Soft deletes with 30-day recovery period

---

This schema supports the MVP and is designed to scale for future features. All timestamps use Firestore Timestamp type. Ensure proper indexing before production deployment.
