# Gamification System

## Overview

Lingsy uses a comprehensive gamification system to drive engagement, retention, and learning effectiveness. The system is inspired by Duolingo but enhanced with additional mechanics.

---

## Core Gamification Elements

### 1. Experience Points (XP)

**Purpose**: Track overall progress and unlock levels

**Earning XP**:
- Complete exercise: **+10 XP** (base)
- Perfect exercise (first try): **+15 XP**
- Complete lesson: **+50 XP** (bonus)
- Perfect lesson (all correct): **+100 XP** (total)
- Daily streak maintained: **+20 XP**
- Achievement unlocked: **+5 to +500 XP** (varies)
- Practice session: **+2 XP** per card
- Review mistakes: **+5 XP** per corrected mistake

**Combo System**:
- 3 correct in a row: **2x XP multiplier**
- 5 correct in a row: **3x XP multiplier**
- 10 correct in a row: **5x XP multiplier**

**XP Requirements per Level**:
```typescript
function calculateXPForLevel(level: number): number {
  // Exponential growth
  return Math.floor(100 * Math.pow(1.5, level - 1));
}

// Examples:
// Level 1: 0 XP
// Level 2: 100 XP
// Level 3: 150 XP
// Level 4: 225 XP
// Level 5: 338 XP
// Level 10: 1,934 XP
// Level 20: 33,306 XP
```

**Display**:
- Progress bar on home screen
- Current level badge (icon + number)
- XP count visible everywhere
- Level-up animation with confetti

---

### 2. Levels

**Purpose**: Long-term progression milestone

**Level Benefits**:
- Unlock new lessons
- Unlock new features
- Social status/bragging rights
- Achievement opportunities

**Level Milestones**:
- **Level 5**: Unlock practice modes
- **Level 10**: Unlock leaderboards
- **Level 15**: Unlock advanced lessons
- **Level 20**: Unlock mastery challenges
- **Level 25**: Unlock social features

**Level Icons**:
- 1-5: Bronze badge
- 6-10: Silver badge
- 11-20: Gold badge
- 21-30: Platinum badge
- 31-50: Diamond badge
- 51+: Legendary badge

**Level-Up Rewards**:
- Confetti animation
- Sound effect
- **+50 gems** bonus
- Achievement check
- Social share option

---

### 3. Hearts (Health Points)

**Purpose**: Limit unlimited retries, encourage focused learning, monetization

**Mechanics**:
- **Max hearts**: 5
- **Starting hearts**: 5
- **Cost per wrong answer**: 1 heart
- **Regeneration**: 1 heart per 30 minutes (automatic)
- **Full regeneration**: 2.5 hours

**Heart States**:
- 5 hearts: Full (green)
- 3-4 hearts: Good (yellow)
- 1-2 hearts: Low (orange)
- 0 hearts: Empty (red) - can't start lessons

**Losing Hearts**:
- Wrong answer in lesson: -1 heart
- Practice mode: No hearts lost
- Flashcard review: No hearts lost

**Refill Options**:
1. **Wait** (free): Timer shows next heart
2. **Watch ad** (future): +1 heart per ad (max 1/day)
3. **Spend gems**: 50 gems = full refill (5 hearts)
4. **Premium subscription**: Unlimited hearts

**Display**:
- Top-right corner of all screens
- Heart icons with fill state
- Timer underneath when < 5
- Pulse animation when at 0

**Notifications**:
- Push notification when hearts full (if enabled)
- "Hearts refilled!" message

---

### 4. Gems (Premium Currency)

**Purpose**: Monetization, rewards, soft premium currency

**Earning Gems** (free):
- First-time lesson completion: **+10 gems**
- Perfect lesson: **+5 gems**
- Level up: **+50 gems**
- Achievement unlock: **+5 to +100 gems**
- Daily login (7-day streak): **+20 gems**
- Invite friend who completes lesson: **+100 gems**
- Premium monthly allowance: **+500 gems**

**Spending Gems**:
- Refill hearts (5 hearts): **50 gems**
- Hint during exercise: **10 gems**
- Skip exercise (future): **20 gems**
- Unlock cosmetic avatar: **100-500 gems**
- Boost XP (2x for 1 hour): **200 gems**

**Purchasing Gems**:
- 100 gems: **$0.99**
- 500 gems: **$3.99** (20% bonus = 600 total)
- 1200 gems: **$7.99** (50% bonus = 1800 total)
- 3000 gems: **$14.99** (100% bonus = 6000 total)

**Display**:
- Top-right corner with gem icon
- Animated sparkle when earned
- Shop icon accessible everywhere

---

### 5. Streaks

**Purpose**: Daily habit formation, retention

**Mechanics**:
- **Streak requirement**: Complete at least 1 lesson per day
- **Timezone**: User's local timezone
- **Grace period**: None (strict 24-hour window)
- **Streak freeze** (future): Use 100 gems to protect 1 day

**Tracking**:
- Current streak (days)
- Longest streak (all-time record)
- Last lesson date

**Streak Milestones**:
- 3 days: Achievement + 20 gems
- 7 days: Achievement + 50 gems
- 14 days: Achievement + 100 gems
- 30 days: Achievement + 200 gems
- 100 days: Achievement + 500 gems
- 365 days: Achievement + 1000 gems

**Display**:
- Flame icon 🔥 with number
- Color changes:
  - 1-6 days: Orange flame
  - 7-29 days: Red flame
  - 30-99 days: Purple flame
  - 100+ days: Rainbow flame

**Notifications**:
- Daily reminder at set time
- "Don't break your streak!" if not completed by 8 PM
- Celebration when milestone reached

**Streak Loss**:
- Sad animation
- "You lost your X-day streak" message
- Encourage to start again
- Show longest streak for motivation

---

### 6. Achievements

**Purpose**: Goals, milestones, engagement, collection

**Categories**:

#### Learning Achievements
- **First Lesson**: Complete your first lesson (5 XP, 10 gems)
- **Quick Learner**: Complete 10 lessons (25 XP, 25 gems)
- **Dedicated Student**: Complete 50 lessons (100 XP, 100 gems)
- **Scholar**: Complete 100 lessons (250 XP, 250 gems)
- **Perfect Start**: Get all answers correct in first lesson (10 XP, 20 gems)
- **Perfectionist**: Complete 10 perfect lessons (50 XP, 50 gems)
- **Flawless**: Complete 50 perfect lessons (200 XP, 200 gems)
- **Word Master**: Learn 100 words (50 XP, 50 gems)
- **Vocabulary King**: Learn 500 words (200 XP, 200 gems)
- **Polyglot Potential**: Learn 1000 words (500 XP, 500 gems)

#### Streak Achievements
- **Getting Started**: 3-day streak (10 XP, 20 gems)
- **Week Warrior**: 7-day streak (25 XP, 50 gems)
- **Dedicated**: 14-day streak (50 XP, 100 gems)
- **Unstoppable**: 30-day streak (100 XP, 200 gems)
- **Legendary**: 100-day streak (500 XP, 1000 gems)
- **Master of Habit**: 365-day streak (1000 XP, 5000 gems)
- **Weekend Warrior**: Complete lessons on Sat & Sun (15 XP, 30 gems)

#### Social Achievements (Future)
- **Social Butterfly**: Add 5 friends (10 XP, 20 gems)
- **Competitive**: Win 10 friend challenges (50 XP, 100 gems)
- **Top Dog**: Reach #1 on friends leaderboard (100 XP, 200 gems)
- **League Champion**: Finish #1 in league (200 XP, 500 gems)

#### Special Achievements
- **Early Bird**: Complete lesson before 7 AM (10 XP, 20 gems)
- **Night Owl**: Complete lesson after 10 PM (10 XP, 20 gems)
- **Speed Demon**: Complete lesson in under 3 minutes (25 XP, 50 gems)
- **Comeback Kid**: Return after 30+ day absence (50 XP, 100 gems)
- **Early Adopter**: Join during first month (100 XP, 200 gems)
- **Supporter**: Purchase gems (50 XP, 0 gems)
- **Premium Member**: Subscribe to premium (100 XP, 500 gems)

**Achievement Properties**:
```typescript
interface Achievement {
  id: string;
  title: LocalizedString;
  description: LocalizedString;
  category: 'learning' | 'streaks' | 'social' | 'special';
  rarity: 'common' | 'rare' | 'epic' | 'legendary';
  xpReward: number;
  gemsReward: number;
  isSecret: boolean;          // Hidden until unlocked
  iconUrl: string;
  iconUrlLocked: string;      // Grayscale
}
```

**Display**:
- Grid view with progress bars
- Filter by category
- Locked achievements in grayscale
- Unlock animation with confetti
- Share to social media

---

### 7. Leaderboards

**Purpose**: Social competition, motivation

**Types**:

#### Weekly Global Leaderboard
- Top 50 users worldwide
- Reset every Monday 00:00 UTC
- Ranked by XP earned this week
- Shows rank, username, avatar, XP

#### Friends Leaderboard
- All added friends
- Ranked by XP this week
- Shows position change (↑↓→)
- "Add friends" CTA if empty

#### League System (Future)
- 10 leagues: Bronze → Silver → Gold → Sapphire → Ruby → Emerald → Diamond → Obsidian → Pearl → Legendary
- 50 users per league
- Top 10 promoted, bottom 5 demoted
- Weekly competition Monday-Sunday
- Rewards for top 3 finishers

**Display**:
- Tab view (Friends / Global / League)
- User's position pinned at top if not in top 10
- Avatar, username, XP, rank
- Medal icons for top 3 (🥇🥈🥉)

**Privacy**:
- Option to hide from global leaderboard
- Always visible to friends
- Anonymize option (username hidden)

---

### 8. Progress Tracking

**Purpose**: Visualize advancement

**Metrics Tracked**:

#### Daily
- Lessons completed today
- XP earned today
- Time spent today
- Current streak status
- Accuracy today

#### Weekly
- Lessons completed this week
- XP earned (for leaderboard)
- Days active
- Average accuracy
- Longest session

#### All-Time
- Total lessons completed
- Total XP earned
- Current level
- Words learned
- Accuracy rate
- Longest streak
- Time spent learning
- Achievements unlocked

**Visualizations**:
- XP chart (bar graph, 7/30 days)
- Accuracy over time (line graph)
- Time spent (area chart)
- Lessons per day (bar chart)
- Heatmap calendar (like GitHub)

---

### 9. Daily Goals

**Purpose**: Daily motivation, habit building

**Mechanics**:
- User sets daily XP goal (50/100/200/500)
- Progress bar on home screen
- Celebration when achieved
- Bonus: +20 gems for 7-day goal streak

**Goals**:
- **Casual**: 50 XP (~1 lesson)
- **Regular**: 100 XP (~2 lessons)
- **Serious**: 200 XP (~4 lessons)
- **Intense**: 500 XP (~10 lessons)

**Display**:
- Circular progress indicator
- Current XP / Goal XP
- Completion percentage
- Encouraging messages

**Rewards**:
- Visual celebration
- "Daily goal achieved!" badge
- Gems if 7 consecutive days

---

### 10. Mastery System

**Purpose**: Long-term retention, spaced repetition

**Levels**:
0. **New**: Just learned
1. **Familiar**: Seen 2-3 times
2. **Comfortable**: Correct 3+ times
3. **Proficient**: Correct 5+ times
4. **Advanced**: Correct 10+ times
5. **Mastered**: Correct 20+ times, no mistakes in 30 days

**Spaced Repetition (Leitner System)**:
- **Box 1**: Review every day
- **Box 2**: Review every 3 days
- **Box 3**: Review every week
- **Box 4**: Review every 2 weeks
- **Box 5**: Review every month

**Mechanics**:
- Correct answer: Move to next box
- Incorrect answer: Move back to Box 1
- Schedule next review based on box

**Display**:
- Mastery badges on vocabulary
- Progress ring (0-5 levels)
- Color coding (red → green)
- "Words to review" counter

---

## Gamification Formulas

### XP to Next Level
```typescript
function xpForLevel(level: number): number {
  return Math.floor(100 * Math.pow(1.5, level - 1));
}
```

### Stars Rating (per lesson)
```typescript
function calculateStars(accuracy: number): 0 | 1 | 2 | 3 {
  if (accuracy < 60) return 0;
  if (accuracy < 80) return 1;
  if (accuracy < 95) return 2;
  return 3;
}
```

### Heart Regeneration
```typescript
function nextHeartTime(lastRegenTime: Date, currentHearts: number): Date {
  if (currentHearts >= 5) return null;
  const minutesPer Heart = 30;
  return new Date(lastRegenTime.getTime() + minutesPer Heart * 60 * 1000);
}
```

### Streak Validation
```typescript
function isStreakMaintained(lastLessonDate: Date, now: Date): boolean {
  const hoursSinceLastLesson = (now.getTime() - lastLessonDate.getTime()) / (1000 * 60 * 60);
  return hoursSinceLastLesson < 24;
}
```

---

## Engagement Hooks

### Daily Login Rewards
- Day 1: 10 gems
- Day 2: 15 gems
- Day 3: 20 gems
- Day 4: 25 gems
- Day 5: 30 gems
- Day 6: 35 gems
- Day 7: 100 gems + bonus

### Push Notifications
- **Daily reminder**: "Time to practice! Your streak is waiting 🔥"
- **Streak warning**: "Don't lose your 7-day streak! 2 hours left ⏰"
- **Hearts refilled**: "Your hearts are full! Ready to learn? ❤️"
- **Achievement unlocked**: "You unlocked [Achievement]! 🏆"
- **Friend challenge**: "[Friend] challenged you! 🎯"
- **Leaderboard**: "You're #3 this week! Can you reach #1? 🏅"

### Celebrations
- **Lesson complete**: Confetti + sound
- **Level up**: Full-screen animation + fanfare
- **Achievement**: Pop-up + badge reveal
- **Perfect lesson**: Extra confetti + gems rain
- **Streak milestone**: Fireworks + special message

---

## Balancing

### Free vs Premium

**Free Users**:
- 5 hearts (regenerating)
- Can earn ~50 gems/day actively
- All core lessons accessible
- Basic achievements

**Premium Users**:
- Unlimited hearts
- 500 gems/month allowance
- Early access to new features
- Exclusive achievements
- Priority support
- Offline lessons

### Gem Economy
- **Earning rate**: ~50-100 gems/day (free)
- **Spending rate**: ~50 gems/day (heart refills)
- **Balance**: Free users can sustain with careful play
- **Incentive**: Power users need to purchase or subscribe

### Difficulty Curve
- Lessons 1-10: Easy (90%+ accuracy expected)
- Lessons 11-30: Medium (80%+ accuracy expected)
- Lessons 31-50: Challenging (70%+ accuracy expected)
- Lessons 51+: Advanced (60%+ accuracy expected)

---

## Analytics to Track

- Daily Active Users (DAU)
- Weekly Active Users (WAU)
- Monthly Active Users (MAU)
- D1 / D7 / D30 Retention
- Lessons per session
- Session length
- Streak distribution
- Heart usage patterns
- Gem earning vs spending
- Conversion to premium
- Achievement unlock rates
- Leaderboard engagement
- Social feature adoption

---

## Future Enhancements

- Clubs/Teams (group learning)
- Tournaments (time-limited competitions)
- Seasonal events (holiday themes)
- Limited-time achievements
- Cosmetic customization (avatars, themes)
- Power-ups (boosters, multipliers)
- Daily challenges (special quests)
- Mini-games (word puzzles, etc.)

---

This gamification system is designed to maximize engagement while maintaining a healthy free-to-premium conversion funnel. All mechanics should be A/B tested before full rollout.
