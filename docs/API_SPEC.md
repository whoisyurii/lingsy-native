# API Specification

## Overview

Lingsy uses Firebase Cloud Functions as the serverless backend. The mobile app communicates with Firebase services directly, and Cloud Functions handle business logic, webhooks, and scheduled tasks.

---

## Cloud Functions

### Base URL
- **Development**: `https://us-central1-lingsy-dev.cloudfunctions.net`
- **Production**: `https://us-central1-lingsy-prod.cloudfunctions.net`

---

## Authentication Functions

### 1. onUserCreate (Trigger)

**Type**: Firebase Auth Trigger
**Event**: `functions.auth.user().onCreate()`

**Purpose**: Initialize user data when account is created

**Trigger**: Automatic when user signs up

**Logic**:
```typescript
export const onUserCreate = functions.auth.user().onCreate(async (user) => {
  const userId = user.uid
  const email = user.email

  // Create user document
  await firestore().collection('users').doc(userId).set({
    uid: userId,
    email: email,
    username: email.split('@')[0],
    displayName: '',
    avatarUrl: null,

    // Language
    interfaceLanguage: 'en',
    learningLanguage: 'en',

    // Gamification
    level: 1,
    xp: 0,
    xpToNextLevel: 100,
    gems: 0,
    hearts: 5,
    lastHeartRegenTime: admin.firestore.FieldValue.serverTimestamp(),

    // Streaks
    currentStreak: 0,
    longestStreak: 0,
    lastLessonDate: null,

    // Stats
    totalLessonsCompleted: 0,
    totalExercisesCompleted: 0,
    accuracyRate: 0,
    totalTimeSpentMinutes: 0,
    wordsLearned: 0,

    // Premium
    isPremium: false,

    // Settings (defaults)
    settings: {
      notifications: {
        dailyReminder: true,
        dailyReminderTime: '09:00',
        streakReminder: true,
        achievementAlerts: true,
      },
      audio: {
        soundEffects: true,
        autoplay: true,
        volume: 1.0,
      },
      haptics: true,
      offlineMode: false,
    },

    // Metadata
    createdAt: admin.firestore.FieldValue.serverTimestamp(),
    updatedAt: admin.firestore.FieldValue.serverTimestamp(),
    lastSeenAt: admin.firestore.FieldValue.serverTimestamp(),
  })

  // Initialize achievements
  await initializeUserAchievements(userId)

  // Send welcome email (optional)
  await sendWelcomeEmail(email)
})
```

---

### 2. onUserDelete (Trigger)

**Type**: Firebase Auth Trigger
**Event**: `functions.auth.user().onDelete()`

**Purpose**: Clean up user data when account is deleted (GDPR compliance)

**Trigger**: Automatic when user account is deleted

**Logic**:
```typescript
export const onUserDelete = functions.auth.user().onDelete(async (user) => {
  const userId = user.uid
  const batch = firestore().batch()

  // Delete user document
  batch.delete(firestore().collection('users').doc(userId))

  // Delete user progress
  const progressDocs = await firestore()
    .collection('userProgress')
    .where('userId', '==', userId)
    .get()
  progressDocs.forEach((doc) => batch.delete(doc.ref))

  // Delete user achievements
  const achievementDocs = await firestore()
    .collection('userAchievements')
    .where('userId', '==', userId)
    .get()
  achievementDocs.forEach((doc) => batch.delete(doc.ref))

  // Delete user vocabulary
  const vocabDocs = await firestore()
    .collection('userVocabulary')
    .where('userId', '==', userId)
    .get()
  vocabDocs.forEach((doc) => batch.delete(doc.ref))

  // Delete daily stats
  const statsDocs = await firestore()
    .collection('dailyStats')
    .where('userId', '==', userId)
    .get()
  statsDocs.forEach((doc) => batch.delete(doc.ref))

  await batch.commit()
})
```

---

## Lesson & Progress Functions

### 3. onLessonComplete (Trigger)

**Type**: Firestore Trigger
**Event**: `functions.firestore.document('userProgress/{progressId}').onWrite()`

**Purpose**: Update user stats, check achievements, update leaderboard when lesson is completed

**Trigger**: Automatic when userProgress document is updated with `isCompleted: true`

**Logic**:
```typescript
export const onLessonComplete = functions.firestore
  .document('userProgress/{progressId}')
  .onWrite(async (change, context) => {
    const before = change.before.data()
    const after = change.after.data()

    // Only process if newly completed
    if (!before?.isCompleted && after?.isCompleted) {
      const userId = after.userId
      const lessonId = after.lessonId

      // Update user stats
      await firestore().collection('users').doc(userId).update({
        totalLessonsCompleted: admin.firestore.FieldValue.increment(1),
        totalExercisesCompleted: admin.firestore.FieldValue.increment(after.totalAnswers),
        'accuracyRate': calculateNewAccuracy(userId, after.accuracy),
        totalTimeSpentMinutes: admin.firestore.FieldValue.increment(
          Math.floor(after.timeSpentSeconds / 60)
        ),
        wordsLearned: admin.firestore.FieldValue.increment(after.vocabularyIds?.length || 0),
        xp: admin.firestore.FieldValue.increment(after.xpEarned),
        updatedAt: admin.firestore.FieldValue.serverTimestamp(),
      })

      // Check and update streak
      await checkStreak(userId)

      // Update daily stats
      await updateDailyStats(userId, after)

      // Check achievements
      await checkAchievements(userId)

      // Update leaderboard
      await updateUserLeaderboardScore(userId, after.xpEarned)

      // Check if leveled up
      await checkLevelUp(userId)
    }
  })
```

---

### 4. getLesson (HTTP)

**Type**: Callable Function
**Method**: POST
**Endpoint**: `/getLesson`

**Purpose**: Get lesson details with authorization check

**Request**:
```typescript
{
  lessonId: string
}
```

**Response**:
```typescript
{
  lesson: Lesson,
  exercises: Exercise[],
  userProgress?: UserProgress,
  isLocked: boolean,
  lockReason?: string
}
```

**Logic**:
```typescript
export const getLesson = functions.https.onCall(async (data, context) => {
  // Authenticate
  if (!context.auth) {
    throw new functions.https.HttpsError('unauthenticated', 'User must be logged in')
  }

  const userId = context.auth.uid
  const { lessonId } = data

  // Get lesson
  const lessonDoc = await firestore().collection('lessons').doc(lessonId).get()
  if (!lessonDoc.exists) {
    throw new functions.https.HttpsError('not-found', 'Lesson not found')
  }

  const lesson = lessonDoc.data()

  // Check if locked
  const user = await firestore().collection('users').doc(userId).get()
  const userData = user.data()

  let isLocked = false
  let lockReason = ''

  // Check level requirement
  if (lesson.requiredLevel > userData.level) {
    isLocked = true
    lockReason = `Requires level ${lesson.requiredLevel}`
  }

  // Check prerequisite lessons
  if (lesson.prerequisiteLessonIds?.length > 0) {
    const prerequisitesComplete = await checkPrerequisites(
      userId,
      lesson.prerequisiteLessonIds
    )
    if (!prerequisitesComplete) {
      isLocked = true
      lockReason = 'Complete previous lessons first'
    }
  }

  // Get exercises
  const exercisesSnapshot = await firestore()
    .collection('exercises')
    .where('lessonId', '==', lessonId)
    .orderBy('order', 'asc')
    .get()

  const exercises = exercisesSnapshot.docs.map((doc) => ({
    id: doc.id,
    ...doc.data(),
  }))

  // Get user progress
  const progressDoc = await firestore()
    .collection('userProgress')
    .doc(`${userId}_${lessonId}`)
    .get()

  return {
    lesson: { id: lessonDoc.id, ...lesson },
    exercises,
    userProgress: progressDoc.exists ? progressDoc.data() : null,
    isLocked,
    lockReason,
  }
})
```

---

## Achievement Functions

### 5. checkAchievements (Background)

**Type**: Background Function (called by other functions)

**Purpose**: Check if user has unlocked any achievements

**Logic**:
```typescript
async function checkAchievements(userId: string) {
  const user = await firestore().collection('users').doc(userId).get()
  const userData = user.data()

  const achievements = await firestore().collection('achievements').get()

  for (const achievementDoc of achievements.docs) {
    const achievement = achievementDoc.data()
    const achievementId = achievementDoc.id

    const userAchievementDoc = await firestore()
      .collection('userAchievements')
      .doc(`${userId}_${achievementId}`)
      .get()

    const userAchievement = userAchievementDoc.data()

    // Skip if already unlocked
    if (userAchievement?.isUnlocked) continue

    // Check if requirement met
    const isMet = await checkAchievementRequirement(
      userId,
      userData,
      achievement
    )

    if (isMet) {
      // Unlock achievement
      await firestore()
        .collection('userAchievements')
        .doc(`${userId}_${achievementId}`)
        .update({
          isUnlocked: true,
          progress: achievement.requirement.value,
          percentage: 100,
          xpEarned: achievement.xpReward,
          gemsEarned: achievement.gemsReward,
          unlockedAt: admin.firestore.FieldValue.serverTimestamp(),
          updatedAt: admin.firestore.FieldValue.serverTimestamp(),
        })

      // Grant rewards
      await firestore().collection('users').doc(userId).update({
        xp: admin.firestore.FieldValue.increment(achievement.xpReward),
        gems: admin.firestore.FieldValue.increment(achievement.gemsReward),
      })

      // Send push notification
      await sendAchievementNotification(userId, achievement)
    } else {
      // Update progress
      const progress = calculateAchievementProgress(userData, achievement)
      await firestore()
        .collection('userAchievements')
        .doc(`${userId}_${achievementId}`)
        .update({
          progress,
          percentage: (progress / achievement.requirement.value) * 100,
          updatedAt: admin.firestore.FieldValue.serverTimestamp(),
        })
    }
  }
}
```

---

## Payment Functions

### 6. revenuecatWebhook (HTTP)

**Type**: HTTP Function
**Method**: POST
**Endpoint**: `/revenuecatWebhook`

**Purpose**: Handle payment events from RevenueCat

**Request** (from RevenueCat):
```typescript
{
  event: {
    type: 'INITIAL_PURCHASE' | 'RENEWAL' | 'CANCELLATION' | 'NON_RENEWING_PURCHASE' | ...,
    app_user_id: string,
    product_id: string,
    price: number,
    currency: string,
    transaction_id: string,
    // ... other fields
  }
}
```

**Response**:
```typescript
200 OK
```

**Logic**:
```typescript
export const revenuecatWebhook = functions.https.onRequest(async (req, res) => {
  const event = req.body.event

  try {
    switch (event.type) {
      case 'INITIAL_PURCHASE':
      case 'RENEWAL':
        await handleSubscriptionStart(event)
        break

      case 'CANCELLATION':
        await handleSubscriptionCancel(event)
        break

      case 'EXPIRATION':
        await handleSubscriptionExpire(event)
        break

      case 'NON_RENEWING_PURCHASE':
        await handleGemPurchase(event)
        break

      default:
        console.log(`Unhandled event type: ${event.type}`)
    }

    res.sendStatus(200)
  } catch (error) {
    console.error('RevenueCat webhook error:', error)
    res.status(500).send('Internal server error')
  }
})

async function handleSubscriptionStart(event: RevenueCatEvent) {
  const userId = event.app_user_id

  await firestore().collection('users').doc(userId).update({
    isPremium: true,
    premiumExpiresAt: new Date(event.expiration_at_ms),
    revenueCatUserId: event.app_user_id,
    updatedAt: admin.firestore.FieldValue.serverTimestamp(),
  })

  // Grant 500 monthly gems
  await firestore().collection('users').doc(userId).update({
    gems: admin.firestore.FieldValue.increment(500),
  })

  // Log transaction
  await logTransaction(userId, 'subscription', event)

  // Send welcome email
  await sendPremiumWelcomeEmail(userId)
}

async function handleGemPurchase(event: RevenueCatEvent) {
  const userId = event.app_user_id
  const productId = event.product_id

  // Map product ID to gem amount
  const gemAmounts: Record<string, number> = {
    gems_100: 100,
    gems_600: 600,   // Includes 20% bonus
    gems_1800: 1800, // Includes 50% bonus
    gems_6000: 6000, // Includes 100% bonus
  }

  const gemsToGrant = gemAmounts[productId] || 0

  if (gemsToGrant > 0) {
    await firestore().collection('users').doc(userId).update({
      gems: admin.firestore.FieldValue.increment(gemsToGrant),
      updatedAt: admin.firestore.FieldValue.serverTimestamp(),
    })

    // Log transaction
    await logTransaction(userId, 'gems_purchase', event)
  }
}
```

---

## Leaderboard Functions

### 7. updateLeaderboards (Scheduled)

**Type**: Scheduled Function
**Schedule**: Every hour
**Cron**: `0 * * * *`

**Purpose**: Update leaderboard snapshots

**Logic**:
```typescript
export const updateLeaderboards = functions.pubsub
  .schedule('0 * * * *')
  .timeZone('UTC')
  .onRun(async (context) => {
    await updateGlobalLeaderboard()
    await updateCountryLeaderboards()
    await updateLeagues()
  })

async function updateGlobalLeaderboard() {
  const currentWeek = getWeekString() // e.g., "2024-W47"

  // Get top 100 users by XP this week
  const usersSnapshot = await firestore()
    .collection('users')
    .orderBy('xp', 'desc')
    .limit(100)
    .get()

  const entries: LeaderboardEntry[] = []

  for (const userDoc of usersSnapshot.docs) {
    const userData = userDoc.data()

    // Get XP earned this week
    const weeklyXP = await getWeeklyXP(userDoc.id, currentWeek)

    entries.push({
      userId: userDoc.id,
      username: userData.username,
      avatarUrl: userData.avatarUrl,
      xp: weeklyXP,
      rank: entries.length + 1,
    })
  }

  // Save leaderboard snapshot
  await firestore()
    .collection('leaderboards')
    .doc(`${currentWeek}_global`)
    .set({
      type: 'global',
      period: currentWeek,
      startDate: getWeekStartDate(currentWeek),
      endDate: getWeekEndDate(currentWeek),
      entries,
      lastUpdated: admin.firestore.FieldValue.serverTimestamp(),
      isActive: true,
    })
}
```

---

### 8. resetWeeklyLeaderboards (Scheduled)

**Type**: Scheduled Function
**Schedule**: Every Monday at 00:00 UTC
**Cron**: `0 0 * * 1`

**Purpose**: Reset weekly leaderboards

**Logic**:
```typescript
export const resetWeeklyLeaderboards = functions.pubsub
  .schedule('0 0 * * 1')
  .timeZone('UTC')
  .onRun(async (context) => {
    const lastWeek = getLastWeekString()

    // Mark last week's leaderboards as inactive
    const leaderboardsSnapshot = await firestore()
      .collection('leaderboards')
      .where('period', '==', lastWeek)
      .get()

    const batch = firestore().batch()
    leaderboardsSnapshot.forEach((doc) => {
      batch.update(doc.ref, { isActive: false })
    })

    await batch.commit()

    // Send rewards to top 3 (future feature)
    await rewardTopPlayers(lastWeek)
  })
```

---

## Streak Functions

### 9. checkStreaks (Scheduled)

**Type**: Scheduled Function
**Schedule**: Every day at 00:01 UTC
**Cron**: `1 0 * * *`

**Purpose**: Check all users' streaks and reset if broken

**Logic**:
```typescript
export const checkStreaks = functions.pubsub
  .schedule('1 0 * * *')
  .timeZone('UTC')
  .onRun(async (context) => {
    const yesterday = new Date()
    yesterday.setDate(yesterday.getDate() - 1)
    yesterday.setHours(0, 0, 0, 0)

    const usersSnapshot = await firestore()
      .collection('users')
      .where('currentStreak', '>', 0)
      .get()

    const batch = firestore().batch()

    for (const userDoc of usersSnapshot.docs) {
      const userData = userDoc.data()
      const lastLessonDate = userData.lastLessonDate?.toDate()

      // Check if user completed a lesson yesterday
      if (!lastLessonDate || lastLessonDate < yesterday) {
        // Streak broken
        batch.update(userDoc.ref, {
          currentStreak: 0,
          updatedAt: admin.firestore.FieldValue.serverTimestamp(),
        })

        // Send "you lost your streak" notification
        await sendStreakLostNotification(userDoc.id, userData.currentStreak)
      }
    }

    await batch.commit()
  })
```

---

### 10. sendStreakReminders (Scheduled)

**Type**: Scheduled Function
**Schedule**: Every day at 20:00 UTC
**Cron**: `0 20 * * *`

**Purpose**: Remind users to maintain their streak

**Logic**:
```typescript
export const sendStreakReminders = functions.pubsub
  .schedule('0 20 * * *')
  .timeZone('UTC')
  .onRun(async (context) => {
    const today = new Date()
    today.setHours(0, 0, 0, 0)

    // Find users with active streaks who haven't completed a lesson today
    const usersSnapshot = await firestore()
      .collection('users')
      .where('currentStreak', '>', 0)
      .where('settings.notifications.streakReminder', '==', true)
      .get()

    for (const userDoc of usersSnapshot.docs) {
      const userData = userDoc.data()
      const lastLessonDate = userData.lastLessonDate?.toDate()

      if (!lastLessonDate || lastLessonDate < today) {
        // User hasn't completed a lesson today
        await sendPushNotification(userDoc.id, {
          title: "Don't break your streak! 🔥",
          body: `You have ${userData.currentStreak} days. Complete a lesson to keep it going!`,
          data: { type: 'streak_reminder' },
        })
      }
    }
  })
```

---

## Analytics Functions

### 11. aggregateDailyStats (Scheduled)

**Type**: Scheduled Function
**Schedule**: Every day at 00:00 UTC
**Cron**: `0 0 * * *`

**Purpose**: Aggregate yesterday's stats for all users

**Logic**:
```typescript
export const aggregateDailyStats = functions.pubsub
  .schedule('0 0 * * *')
  .timeZone('UTC')
  .onRun(async (context) => {
    const yesterday = getYesterdayString() // "2024-11-17"

    const usersSnapshot = await firestore().collection('users').get()

    for (const userDoc of usersSnapshot.docs) {
      const userId = userDoc.id

      // Get yesterday's activity
      const progressSnapshot = await firestore()
        .collection('userProgress')
        .where('userId', '==', userId)
        .where('completedAt', '>=', getStartOfDay(yesterday))
        .where('completedAt', '<', getEndOfDay(yesterday))
        .get()

      let lessonsCompleted = 0
      let exercisesCompleted = 0
      let correctAnswers = 0
      let totalAnswers = 0
      let timeSpentSeconds = 0
      let xpEarned = 0

      progressSnapshot.forEach((doc) => {
        const data = doc.data()
        lessonsCompleted++
        exercisesCompleted += data.totalAnswers
        correctAnswers += data.correctAnswers
        totalAnswers += data.totalAnswers
        timeSpentSeconds += data.timeSpentSeconds
        xpEarned += data.xpEarned
      })

      const accuracy = totalAnswers > 0 ? (correctAnswers / totalAnswers) * 100 : 0

      // Save daily stats
      await firestore()
        .collection('dailyStats')
        .doc(`${userId}_${yesterday}`)
        .set({
          userId,
          date: yesterday,
          lessonsCompleted,
          exercisesCompleted,
          timeSpentMinutes: Math.floor(timeSpentSeconds / 60),
          correctAnswers,
          totalAnswers,
          accuracy,
          xpEarned,
          gemsEarned: 0, // TODO: track gem earnings
          maintainedStreak: lessonsCompleted > 0,
          createdAt: admin.firestore.FieldValue.serverTimestamp(),
          updatedAt: admin.firestore.FieldValue.serverTimestamp(),
        })
    }
  })
```

---

## Admin Functions

### 12. createLesson (HTTP, Admin)

**Type**: Callable Function (Admin only)
**Method**: POST
**Endpoint**: `/createLesson`

**Purpose**: Create a new lesson (admin tool)

**Request**:
```typescript
{
  title: { en: string, uk: string },
  description: { en: string, uk: string },
  order: number,
  level: number,
  exercises: Exercise[],
  // ... other fields
}
```

**Response**:
```typescript
{
  lessonId: string
}
```

**Logic**:
```typescript
export const createLesson = functions.https.onCall(async (data, context) => {
  // Check admin role
  if (!context.auth || !context.auth.token.admin) {
    throw new functions.https.HttpsError(
      'permission-denied',
      'Must be an admin to create lessons'
    )
  }

  const { title, description, order, level, exercises, ...rest } = data

  // Create lesson
  const lessonRef = await firestore().collection('lessons').add({
    title,
    description,
    order,
    level,
    exerciseIds: [],
    isPublished: false,
    createdAt: admin.firestore.FieldValue.serverTimestamp(),
    updatedAt: admin.firestore.FieldValue.serverTimestamp(),
    createdBy: context.auth.uid,
    ...rest,
  })

  // Create exercises
  const exerciseIds: string[] = []

  for (const exercise of exercises) {
    const exerciseRef = await firestore().collection('exercises').add({
      ...exercise,
      lessonId: lessonRef.id,
      createdAt: admin.firestore.FieldValue.serverTimestamp(),
      updatedAt: admin.firestore.FieldValue.serverTimestamp(),
    })

    exerciseIds.push(exerciseRef.id)
  }

  // Update lesson with exercise IDs
  await lessonRef.update({ exerciseIds })

  return { lessonId: lessonRef.id }
})
```

---

## Helper Functions

### Heart Regeneration

```typescript
export async function regenerateHearts(userId: string) {
  const userDoc = await firestore().collection('users').doc(userId).get()
  const userData = userDoc.data()

  if (userData.hearts >= 5) return // Already full

  const now = new Date()
  const lastRegen = userData.lastHeartRegenTime.toDate()
  const minutesSinceLastRegen = (now.getTime() - lastRegen.getTime()) / (1000 * 60)

  const heartsToAdd = Math.floor(minutesSinceLastRegen / 30)

  if (heartsToAdd > 0) {
    const newHearts = Math.min(userData.hearts + heartsToAdd, 5)

    await firestore().collection('users').doc(userId).update({
      hearts: newHearts,
      lastHeartRegenTime: admin.firestore.FieldValue.serverTimestamp(),
    })
  }
}
```

---

## Rate Limiting

All HTTP functions should implement rate limiting:

```typescript
import rateLimit from 'express-rate-limit'

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Limit each IP to 100 requests per windowMs
})

export const myFunction = functions.https.onRequest((req, res) => {
  limiter(req, res, () => {
    // Function logic
  })
})
```

---

## Error Handling

All functions should use consistent error handling:

```typescript
try {
  // Function logic
} catch (error) {
  console.error('Error in function:', error)

  if (error.code === 'permission-denied') {
    throw new functions.https.HttpsError('permission-denied', error.message)
  }

  throw new functions.https.HttpsError('internal', 'An error occurred')
}
```

---

This API specification covers all major Cloud Functions needed for the MVP. All functions should be thoroughly tested with Firebase emulators before deployment to production.
