# Lingsy - Language Learning App

## Project Overview

Lingsy is a modern, gamified language learning mobile application built with React Native and Expo. It aims to be a superior alternative to Duolingo, focusing on engaging learning experiences with a strong emphasis on gamification and user retention.

## Core Mission

Provide an engaging, effective, and fun language learning experience that keeps users motivated through gamification, progress tracking, and social features.

## Tech Stack

### Frontend
- **Framework**: React Native (Expo SDK 54)
- **Navigation**: Expo Router + React Navigation
- **State Management**: MobX-State-Tree (Ignite default)
- **UI Components**: Custom components + React Native built-ins
- **Styling**: React Native StyleSheet with theme system
- **Animations**: Reanimated 4
- **Gestures**: React Native Gesture Handler
- **Internationalization**: i18next + react-i18next
- **Local Storage**: MMKV for fast key-value storage

### Backend
- **Primary Backend**: Firebase
  - Firestore: Database
  - Authentication: User management
  - Cloud Functions: Serverless backend logic
  - Storage: Audio files and media
  - Analytics: User behavior tracking

### Payments & Subscriptions
- **RevenueCat**: Subscription management
- **Stripe**: Payment processing
- Both iOS App Store and Google Play billing

### Development
- **Language**: TypeScript
- **Testing**: Jest + React Testing Library + Maestro (E2E)
- **Build**: EAS Build
- **CI/CD**: GitHub Actions (planned)
- **Code Quality**: ESLint + Prettier
- **Dev Tools**: Reactotron

## MVP Scope (Phase 1)

### Languages
- **Learning Language**: English only
- **Interface Languages**: English, Ukrainian

### Core Features
1. **Flashcards**
   - Front: Ukrainian word/phrase
   - Back: English translation
   - Images/icons for visual learning
   - Audio pronunciation

2. **Listening Exercises**
   - Audio playback of English words/phrases
   - Multiple choice answers
   - Repeat functionality
   - Speed control (normal, slow)

3. **Writing Exercises (Write to Match)**
   - Listen to English word/phrase
   - Type the correct answer
   - Keyboard input with validation
   - Hints available

### Gamification (MVP)
- **Health Points System**: Limited lives per day
- **Experience Points (XP)**: Progress tracking
- **Streaks**: Daily learning streaks
- **Levels**: User progression system
- **Basic Achievements**: First lesson, 7-day streak, etc.

### Monetization (MVP)
- **Free Tier**: Limited daily lessons
- **Premium Subscription**: Unlimited access
- **Gems**: Premium currency (purchasable)
- **Hearts**: Health points (regenerate over time)

### User Features
- User authentication (email/password)
- Profile management
- Progress tracking
- Basic statistics

## Future Expansion (Post-MVP)

### Languages
- Spanish
- German
- Portuguese
- French
- Ukrainian (as learning language)

### Advanced Features
- Speaking/pronunciation practice
- Conversation practice
- Stories and reading comprehension
- Grammar explanations
- Community features
- Leaderboards
- Friend challenges
- Offline mode enhancement

## App Structure

```
src/
├── app/                    # Expo Router screens
│   ├── (auth)/            # Auth flow screens
│   ├── (tabs)/            # Main tab navigation
│   └── _layout.tsx        # Root layout
├── components/            # Reusable components
│   ├── common/           # Basic UI components
│   ├── learning/         # Learning-specific components
│   └── gamification/     # Game elements
├── screens/              # Screen components
├── services/             # API and external services
│   ├── firebase/        # Firebase services
│   ├── api/             # API client
│   └── audio/           # Audio service
├── models/               # MobX-State-Tree models
│   ├── UserStore        # User state
│   ├── LessonStore      # Lesson data
│   ├── ProgressStore    # User progress
│   └── GameStore        # Gamification state
├── utils/                # Utility functions
├── theme/                # Theme and styling
├── i18n/                 # Translations
│   ├── en.ts            # English
│   └── uk.ts            # Ukrainian
└── config/               # App configuration

docs/
├── ARCHITECTURE.md       # Technical architecture
├── SCREENS.md           # Screen specifications
├── FEATURES.md          # Feature specifications
├── DATABASE_SCHEMA.md   # Firestore schema
├── GAMIFICATION.md      # Gamification system
├── MONETIZATION.md      # Revenue model
├── API_SPEC.md          # API endpoints
└── IMPLEMENTATION_ROADMAP.md  # Development phases
```

## Development Principles

1. **User-Centric**: Every feature should enhance learning
2. **Performance First**: Smooth 60fps animations, fast loading
3. **Offline-Ready**: Core features work offline
4. **Accessible**: WCAG 2.1 AA compliance
5. **Scalable**: Easy to add new languages and features
6. **Testable**: Minimum 80% code coverage
7. **Type-Safe**: Full TypeScript coverage
8. **Maintainable**: Clean code, documented, modular

## Key Metrics to Track

- Daily Active Users (DAU)
- Retention Rate (D1, D7, D30)
- Lesson Completion Rate
- Average Session Duration
- Streak Maintenance Rate
- Conversion Rate (Free → Premium)
- Churn Rate
- Net Promoter Score (NPS)

## Design Philosophy

- **Playful but Professional**: Fun without being childish
- **Clear Visual Hierarchy**: Easy to scan and understand
- **Immediate Feedback**: Users know instantly if they're correct
- **Progress Visualization**: Always show advancement
- **Celebratory**: Celebrate wins, small and large
- **Non-Judgmental**: Encourage learning from mistakes

## Competitive Advantages

1. **Better Gamification**: More engaging than Duolingo
2. **Focus on Conversation**: Real-world language skills
3. **Community Features**: Learn with friends
4. **Adaptive Learning**: AI-powered personalization (future)
5. **Superior UX**: Smoother, more intuitive interface
6. **Local Focus**: Strong Ukrainian language support

## Important Notes

- Always consider offline scenarios
- Audio files should be cached locally
- Minimize API calls to reduce costs
- Use Firebase efficiently (query limits, indexes)
- Respect user privacy and data
- Follow app store guidelines strictly
- Ensure compliance with COPPA (if targeting children)
- Implement proper error handling and recovery
- Log user actions for analytics
- A/B test major features before full rollout

## Current Status

- ✅ Project scaffolded with Ignite
- 🟡 Documentation in progress
- ⏳ MVP development not started
- ⏳ Firebase setup pending
- ⏳ Content creation pending

## Quick Commands

```bash
# Development
npm start                 # Start Expo dev server
npm run ios              # Run on iOS simulator
npm run android          # Run on Android emulator

# Testing
npm test                 # Run Jest tests
npm run test:watch       # Watch mode
npm run test:maestro     # E2E tests

# Code Quality
npm run lint             # Lint code
npm run compile          # TypeScript check

# Build
npm run build:ios:prod   # iOS production build
npm run build:android:prod # Android production build
```

## Environment Variables

```
FIREBASE_API_KEY=
FIREBASE_AUTH_DOMAIN=
FIREBASE_PROJECT_ID=
FIREBASE_STORAGE_BUCKET=
FIREBASE_MESSAGING_SENDER_ID=
FIREBASE_APP_ID=

REVENUECAT_API_KEY_IOS=
REVENUECAT_API_KEY_ANDROID=

STRIPE_PUBLISHABLE_KEY=
```

## Useful Resources

- [Expo Docs](https://docs.expo.dev/)
- [Firebase Docs](https://firebase.google.com/docs)
- [RevenueCat Docs](https://www.revenuecat.com/docs)
- [Ignite Docs](https://docs.infinite.red/ignite-cli/)
- [React Navigation](https://reactnavigation.org/)
