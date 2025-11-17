# Monetization Strategy

## Overview

Lingsy uses a freemium model with multiple revenue streams:
1. Premium subscriptions (primary)
2. Gem purchases (secondary)
3. Ads (future, limited)

**Goal**: 5-10% conversion rate to premium within 30 days

---

## Revenue Streams

### 1. Premium Subscription (Primary Revenue)

**Tiers**:

#### Monthly Plan
- **Price**: $9.99/month
- **Billing**: Monthly recurring
- **Features**: All premium features

#### Annual Plan (Recommended)
- **Price**: $59.99/year ($4.99/month)
- **Savings**: 50% vs monthly
- **Billing**: Yearly recurring
- **Features**: All premium features + bonus

**Premium Benefits**:

✅ **Unlimited Hearts**
- No heart system restrictions
- Learn without limits
- Never blocked from lessons

✅ **No Ads** (future)
- Ad-free experience
- Cleaner interface

✅ **Offline Lessons**
- Download lessons for offline use
- Practice anywhere
- Auto-sync when online

✅ **Personalized Learning** (future)
- AI-powered lesson recommendations
- Adaptive difficulty
- Custom learning paths

✅ **Mastery Quizzes**
- Advanced assessment tools
- Detailed progress reports
- Skill certification

✅ **Monthly Gem Allowance**
- 500 gems per month
- ~$5 value
- Stackable (unused gems carry over)

✅ **Exclusive Achievements**
- Premium-only badges
- Special challenges
- Leaderboard flair

✅ **Priority Support**
- Faster response time
- Dedicated support channel
- Feature requests priority

✅ **Early Access**
- New features first
- Beta testing opportunities
- Sneak peeks

✅ **Progress Insights**
- Advanced analytics
- Learning patterns
- Recommendations

**Trial**:
- 7-day free trial (annual plan only)
- No credit card required
- Full premium access
- Auto-converts to paid after trial

**Pricing Strategy**:
- Positioned as "less than coffee per week"
- Annual plan emphasized (higher LTV)
- Family plan (future): $14.99/month for up to 6 users

---

### 2. Gem Purchases (In-App Currency)

**Purpose**: Monetize engaged free users who don't want subscription

**Packages**:

| Package | Gems | Price | Bonus | $/Gem | Value Prop |
|---------|------|-------|-------|-------|------------|
| Starter | 100 | $0.99 | - | $0.0099 | Quick refill |
| Popular | 500 | $3.99 | +100 (20%) | $0.0067 | **Best for regular users** |
| Power | 1200 | $7.99 | +600 (50%) | $0.0044 | Great value |
| Ultimate | 3000 | $14.99 | +3000 (100%) | $0.0025 | **Best value** |

**Gem Uses**:
- Refill hearts: 50 gems (emergency use)
- Exercise hints: 10 gems
- Streak freeze: 100 gems (save 1 day)
- Boost XP (2x for 1 hour): 200 gems
- Skip exercise (future): 20 gems
- Cosmetic items: 100-500 gems

**Earning Gems (Free)**:
- First-time lesson: 10 gems
- Perfect lesson: 5 gems
- Level up: 50 gems
- Achievements: 5-500 gems
- Daily login streak (7 days): 20 gems
- Friend referral: 100 gems
- **Max free earning**: ~50-100 gems/day

**Conversion Funnel**:
1. User runs out of hearts
2. Modal shows options:
   - Wait 2.5 hours (free)
   - Refill with 50 gems
   - **Upgrade to Premium** (highlighted)
3. If no gems, prompt to purchase
4. If repeatedly purchasing gems, suggest subscription (better value)

**Purchase Flow**:
```
Tap "Refill Hearts"
  → Modal opens
    → Option 1: Wait (free)
    → Option 2: Use 50 gems
      → If insufficient gems:
        → "Buy Gems" button
          → Gem shop opens
            → Select package
              → Payment via RevenueCat
                → Success → Hearts refilled
    → Option 3: Get Premium (CTA)
      → Premium modal opens
```

---

### 3. Advertising (Future, Limited)

**Strategy**: Minimal ads for free users, removable with premium

**Ad Types**:

#### Rewarded Video Ads
- **Frequency**: 1 per day max
- **Reward**: 1 heart or 20 gems
- **Placement**: Heart refill modal
- **Length**: 15-30 seconds
- **Networks**: Google AdMob, Facebook Audience Network

#### Interstitial Ads (Carefully)
- **Frequency**: After every 5 lessons (free users only)
- **Skippable**: After 5 seconds
- **Placement**: Between lessons (not mid-lesson)
- **Cap**: Max 3 per day

**No Ads**:
- Premium users: Zero ads
- During active lesson
- Mid-exercise
- On home screen

**Revenue Split**:
- Subscriptions: 70%
- Gem purchases: 25%
- Ads: 5%

---

## Pricing Psychology

### Value Anchoring
- Show annual plan savings prominently (50% off!)
- Display "Most Popular" badge on annual plan
- Show monthly equivalent price ($4.99/month)

### Urgency
- "Limited time: 7-day free trial"
- "Save 50% today only" (first-time offer)
- Trial countdown timer

### Social Proof
- "Join 10,000+ premium learners"
- Star ratings and reviews
- "Top language learning app"

### Feature Comparison Table

| Feature | Free | Premium |
|---------|------|---------|
| Core Lessons | ✅ All | ✅ All |
| Hearts | 5 (regenerating) | ♾️ Unlimited |
| Offline Mode | ❌ | ✅ |
| Ads | Yes (future) | ❌ No Ads |
| Gems/month | Earn only | 500 included |
| Mastery Quizzes | ❌ | ✅ |
| Analytics | Basic | Advanced |
| Support | Standard | Priority |

---

## Implementation: RevenueCat + Stripe

### RevenueCat Setup

**Products** (iOS & Android):
```typescript
// Subscription Products
{
  monthly: {
    identifier: 'lingsy_premium_monthly',
    price: '$9.99',
    billing: 'monthly',
    trial: null,
  },
  annual: {
    identifier: 'lingsy_premium_annual',
    price: '$59.99',
    billing: 'yearly',
    trial: '7 days',
  },
}

// Consumable Products (Gems)
{
  gems_100: { identifier: 'gems_100', price: '$0.99' },
  gems_600: { identifier: 'gems_600', price: '$3.99' },
  gems_1800: { identifier: 'gems_1800', price: '$7.99' },
  gems_6000: { identifier: 'gems_6000', price: '$14.99' },
}
```

**Entitlements**:
```typescript
{
  premium: {
    products: ['lingsy_premium_monthly', 'lingsy_premium_annual'],
  }
}
```

**Integration**:
```typescript
import Purchases from 'react-native-purchases';

// Initialize
await Purchases.configure({ apiKey: REVENUECAT_API_KEY });

// Check subscription status
const customerInfo = await Purchases.getCustomerInfo();
const isPremium = customerInfo.entitlements.active['premium'] !== undefined;

// Purchase subscription
try {
  const { customerInfo } = await Purchases.purchasePackage(annualPackage);
  if (customerInfo.entitlements.active['premium']) {
    // User is now premium
    updateUserPremiumStatus(true);
  }
} catch (e) {
  if (e.userCancelled) {
    // User cancelled
  }
}

// Purchase gems
const { customerInfo } = await Purchases.purchaseProduct('gems_100');
// Grant gems via Cloud Function webhook
```

**Webhooks** (RevenueCat → Firebase Cloud Functions):
```typescript
// Cloud Function
export const revenuecatWebhook = functions.https.onRequest(async (req, res) => {
  const event = req.body.event;

  switch (event.type) {
    case 'INITIAL_PURCHASE':
    case 'RENEWAL':
      // Grant premium access
      await updateUserSubscription(event.app_user_id, true);
      break;

    case 'CANCELLATION':
    case 'EXPIRATION':
      // Revoke premium access
      await updateUserSubscription(event.app_user_id, false);
      break;

    case 'NON_RENEWING_PURCHASE':
      // Grant gems
      await grantGems(event.app_user_id, event.product_id);
      break;
  }

  res.sendStatus(200);
});
```

### Stripe Setup (for web payments, future)

**Products**:
- Lingsy Premium Monthly ($9.99)
- Lingsy Premium Annual ($59.99)

**Payment Link Integration**:
```typescript
import { loadStripe } from '@stripe/stripe-react-native';

const stripe = await loadStripe(STRIPE_PUBLISHABLE_KEY);

// Create checkout session (via Cloud Function)
const { sessionId } = await createCheckoutSession(userId, 'annual');

// Redirect to Stripe Checkout
const { error } = await stripe.redirectToCheckout({ sessionId });
```

---

## Conversion Optimization

### Premium Upsell Triggers

1. **Out of Hearts** (primary)
   - User hits 0 hearts
   - Modal: "Get unlimited hearts with Premium"
   - High conversion moment

2. **Streak at Risk**
   - User about to lose long streak
   - "Save your streak + go unlimited"

3. **After Great Session**
   - Completed 3+ perfect lessons
   - "You're doing great! Go unlimited"

4. **Lesson Locked**
   - Reached level requirement
   - "Unlock all lessons with Premium"

5. **7-Day Active User**
   - Used app 7 days straight
   - Special offer: "You're committed! Save 25%"

6. **Friends Comparison**
   - Friend is premium
   - "Join [Friend] with Premium"

### Discount Strategies

**First-Time Offer**:
- 25% off annual plan
- One-time, expires in 48 hours
- "Welcome offer" badge

**Win-Back Campaigns**:
- Lapsed users (30+ days inactive)
- 50% off first month
- Email + push notification

**Seasonal Promotions**:
- New Year: "New Year, New Language" (20% off)
- Black Friday: (40% off annual)
- Back to School: (30% off)

**Referral Incentive**:
- Referrer: 1 month free premium
- Referee: 25% off first month
- Both get 200 gems

---

## Payment Flows

### Subscription Purchase
```
User taps "Go Premium"
  → Premium modal opens
  → User selects plan (monthly/annual)
  → RevenueCat presents native payment sheet
  → User confirms via FaceID/TouchID/PIN
  → Payment processed
  → RevenueCat webhook fires
  → Cloud Function updates Firestore
  → User.isPremium = true
  → App updates UI
  → Celebration modal
  → "Welcome to Premium!" 🎉
```

### Gem Purchase
```
User taps "Buy Gems"
  → Gem shop modal opens
  → User selects package
  → RevenueCat presents payment sheet
  → Payment processed
  → Webhook fires
  → Cloud Function grants gems
  → Firestore: user.gems += amount
  → App updates UI
  → Success toast
  → Gems counter animates up
```

### Subscription Management
```
User taps "Manage Subscription"
  → RevenueCat.showManagementURL()
  → Opens native subscription management
    (App Store or Google Play)
  → User can:
    - Cancel subscription
    - Change plan
    - Update payment method
  → Changes reflect via webhooks
```

---

## Churn Prevention

### At-Risk User Detection
- Decreasing session frequency
- Haven't completed lesson in 3 days
- Subscription cancellation

### Retention Actions
1. **Push Notification**
   - "We miss you! Your streak is waiting"
   - Personalized based on progress

2. **Email Campaign**
   - Highlight new features
   - Share learning tips
   - Offer limited discount

3. **In-App Messaging**
   - "What can we improve?" survey
   - Feature requests
   - Feedback collection

4. **Win-Back Offer**
   - 50% off next month
   - Free month on resubscribe
   - Exclusive content

### Cancellation Flow
```
User cancels subscription
  → "We're sorry to see you go" modal
  → Survey: "Why are you leaving?"
    - Too expensive
    - Not using enough
    - Missing features
    - Technical issues
    - Other
  → Offer retention discount (33% off 3 months)
  → If still cancels:
    - Confirm cancellation
    - "Access until [end date]"
    - "We'd love to have you back"
  → Follow-up email in 30 days
```

---

## Metrics & KPIs

### Subscription Metrics
- **Conversion Rate**: % users → premium (target: 5-10%)
- **MRR** (Monthly Recurring Revenue): Total monthly revenue
- **ARR** (Annual Recurring Revenue): Total annual revenue
- **ARPU** (Average Revenue Per User): Revenue / total users
- **LTV** (Lifetime Value): Avg revenue per user over lifetime
- **Churn Rate**: % subscribers who cancel (target: <5%/month)
- **CAC** (Customer Acquisition Cost): Marketing spend / new users

### Gem Metrics
- Gems purchased vs earned
- Conversion rate: hearts → gems → subscription
- Gem spending patterns
- Gem balance distribution

### Optimization Targets
- 30-day retention: >40%
- Free → Premium: 7-10%
- Trial → Paid: >60%
- Monthly → Annual upgrade: >30%
- Churn: <5%/month
- LTV:CAC ratio: >3:1

---

## Revenue Projections (Year 1)

**Assumptions**:
- 10,000 downloads/month
- 40% D30 retention → 4,000 MAU
- 8% conversion to premium → 320 paid users
- 70% annual, 30% monthly
- 5% gem purchasers → 200 users × $5 avg = $1,000/month

**Monthly Breakdown**:
- Subscriptions:
  - Annual: 224 × $4.99 = $1,118
  - Monthly: 96 × $9.99 = $959
  - **Total**: $2,077/month
- Gems: $1,000/month
- **Total MRR**: $3,077

**After platform fees (30%)**:
- Net revenue: $2,154/month
- **Annual: ~$25,850**

**Year 1 Growth**:
- Month 1: $300
- Month 3: $900
- Month 6: $1,800
- Month 12: $3,000+

---

## Compliance & Legal

### App Store Guidelines
- Clear pricing display
- Easy subscription management
- Transparent billing
- No misleading claims
- Restore purchases button

### Privacy
- GDPR compliance (EU users)
- CCPA compliance (California)
- Data export on request
- Right to deletion
- Privacy policy link

### Refunds
- App Store/Play Store handles
- No refunds for gems (consumable)
- Subscription refunds case-by-case
- Clear refund policy

### Taxes
- VAT collection (EU)
- Sales tax (US states)
- Handled by Apple/Google

---

## Future Monetization

### Family Plan
- $14.99/month for up to 6 users
- Shared premium benefits
- Individual progress tracking

### Lifetime Premium
- One-time payment: $199.99
- Lifetime access
- All future features
- Limited availability

### B2B/Enterprise
- School licenses
- Corporate training
- Bulk discounts
- Admin dashboard

### Merchandise (far future)
- Branded apparel
- Learning materials
- Physical flashcards

---

This monetization strategy balances user experience with revenue generation. The freemium model allows broad user acquisition while providing clear value for premium conversion. All pricing should be A/B tested before final implementation.
