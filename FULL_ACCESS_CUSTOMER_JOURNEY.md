# Full Access - Vásárlói Út és Technikai Architektúra

## 📋 Amit Ígérünk (Weboldal alapján)

### Full Access ($8.99/hó vagy $49/év)
- ✅ 21 új recept hetente
- ✅ Szervezett bevásárlólisták
- ✅ Printable meal prep guides
- ✅ Seasonal variations library
- ✅ Fresh meal plans every week
- ✅ Complete step-by-step recipes
- ✅ Printable PDFs (kitchen-ready)
- ✅ Meal swap suggestions
- ✅ Access to past weeks' plans
- ✅ Weekly Monday motivation email
- ✅ Cancel anytime

---

## 🛤️ VÁSÁRLÓI ÚT (Customer Journey)

### 1️⃣ DISCOVERY & SIGN UP (Első 60 másodperc)

**Felhasználói Műveletek:**
1. Landol a weboldalon
2. Kitölti a generator formot:
   - Életkor (Age Range)
   - Cél (Goal: Weight Loss / Maintain / Muscle Gain)
   - Főzési idő preferencia
   - Étrendi megszorítások
3. Email cím megadása
4. "Generate My Plan" gomb → **FREE 3-Day Plan**

**Backend Folyamat:**
```
POST /api/v1/generate-plan
{
  "email": "user@example.com",
  "profile": {
    "age_range": "30-40",
    "goal": "weight_loss",
    "cooking_time": "30_min",
    "restrictions": ["dairy_free", "nut_allergy"]
  }
}

RESPONSE:
{
  "plan_id": "abc123",
  "user_id": "usr_xyz789",
  "trial_status": "active",
  "plan": {
    "day_1": {...},
    "day_2": {...},
    "day_3": {...}
  },
  "shopping_list": [...],
  "expires_at": "2025-11-12T00:00:00Z"
}
```

**Amit a Felhasználó Kap:**
- ✅ 3 napos személyre szabott meal plan
- ✅ Bevásárlólista
- ✅ Receptek step-by-step leírással
- ✅ Makró számítás (calories, protein, fat, carbs)
- ✅ Email confirmation

---

### 2️⃣ FREE TRIAL EXPERIENCE (3 nap)

**Amit Látnak:**
- ✅ Teljes 3 napos meal plan
- ✅ Printable PDF verzió
- ⚠️ "Upgrade to unlock 21 more recipes" banner
- ⚠️ "Day 4-7 available with Full Access" lock screen

**Backend:**
```javascript
// Trial experience
if (user.subscription_status === 'trial') {
  // Korlátozott funkciók
  accessibleDays: 3,
  accessibleRecipes: meal_plan.days[0-2],
  canPrint: true (limited to 3 days),
  canSwapMeals: false,
  pastPlansAccess: false,
  weeklyEmailEnabled: false
}
```

**Email Journey (Trial期间):**
- **Day 0 (azonnal):** Welcome email + PDF link
- **Day 1 (24h):** "How's it going?" + Tips
- **Day 2 (48h):** "Upgrade now - Unlock 21 recipes" + Limited time offer
- **Day 3 (72h):** "Last day! Don't lose your progress" + Urgency

---

### 3️⃣ UPGRADE TO FULL ACCESS

**Felhasználói Műveletek:**
1. Kattint "Upgrade to Full Access" gombra
2. Választ subscription típust:
   - Monthly: $8.99/hó
   - Annual: $49/év (BEST VALUE)
3. Fizetés Stripe-on keresztül
4. Instant activation

**Backend Flow:**
```
POST /api/v1/subscriptions/create
{
  "user_id": "usr_xyz789",
  "plan": "monthly", // or "annual"
  "payment_method": "pm_stripe_token"
}

RESPONSE:
{
  "subscription_id": "sub_abc123",
  "status": "active",
  "current_period_end": "2025-12-09",
  "features_unlocked": [
    "full_week_plans",
    "21_recipes_weekly",
    "shopping_lists",
    "meal_swaps",
    "past_plans_access",
    "printable_pdfs",
    "weekly_emails"
  ]
}
```

**Stripe Webhook Handling:**
```javascript
// webhook: customer.subscription.created
async function handleSubscriptionCreated(event) {
  const subscription = event.data.object;

  await db.users.update({
    subscription_id: subscription.id,
    subscription_status: 'active',
    subscription_plan: subscription.metadata.plan,
    current_period_end: subscription.current_period_end,
    upgraded_at: new Date()
  });

  // Unlock all features
  await unlockFullAccess(subscription.customer);

  // Send welcome email
  await sendEmail({
    template: 'full_access_welcome',
    to: user.email,
    data: {
      first_week_plan_url: generateWeeklyPlanUrl(user),
      dashboard_url: getDashboardUrl(user)
    }
  });
}
```

---

### 4️⃣ FULL ACCESS EXPERIENCE (Folyamatos)

#### **Heti Automatikus Folyamat (Minden Hétfő 6:00 AM)**

**1. Meal Plan Generálás:**
```javascript
// Cron job: Every Monday 6:00 AM
async function generateWeeklyMeals() {
  const activeUsers = await db.users.find({
    subscription_status: 'active'
  });

  for (const user of activeUsers) {
    // Személyre szabott 7 napos plan (21 recept)
    const weeklyPlan = await aiMealPlanner.generate({
      user_profile: user.profile,
      preferences: user.preferences,
      past_meals: await getPastMeals(user, weeks: 4), // Avoid repetition
      season: getCurrentSeason(),
      week_number: getWeekNumber()
    });

    // Mentés DB-be
    await db.meal_plans.create({
      user_id: user.id,
      week_start: getMonday(),
      meals: weeklyPlan.meals, // 21 recipes
      shopping_list: weeklyPlan.shopping_list,
      nutritional_summary: weeklyPlan.nutrition,
      generated_at: new Date()
    });

    // PDF generálás
    const pdf = await generatePDF({
      plan: weeklyPlan,
      user: user,
      template: 'weekly_meal_plan'
    });

    await uploadToS3(pdf, `meal-plans/${user.id}/${week}.pdf`);
  }
}
```

**2. Email Kiküldés (Monday Morning Motivation):**
```javascript
async function sendWeeklyEmails(users) {
  for (const user of users) {
    const thisWeekPlan = await getWeekPlan(user, 'current');

    await sendEmail({
      template: 'monday_motivation',
      to: user.email,
      subject: `🌿 Your Mediterranean Week is Ready! (21 New Recipes)`,
      data: {
        user_name: user.name,
        week_highlights: [
          thisWeekPlan.meals[0].name, // Featured meal 1
          thisWeekPlan.meals[7].name, // Featured meal 2
          thisWeekPlan.meals[14].name // Featured meal 3
        ],
        pdf_download_url: getPDFUrl(user, 'current_week'),
        shopping_list_url: getShoppingListUrl(user),
        dashboard_url: getDashboardUrl(user),
        motivational_quote: getRandomQuote()
      }
    });
  }
}
```

**3. Bevásárlólista Organizálás:**
```javascript
function organizeShoppingList(weeklyMeals) {
  const items = extractIngredients(weeklyMeals);

  return {
    categories: [
      {
        name: "Fresh Produce",
        items: [
          { name: "Cherry tomatoes", amount: "500g", meals: ["Mon Lunch", "Wed Dinner"] },
          { name: "Spinach", amount: "300g", meals: ["Tue Breakfast", "Thu Dinner"] }
        ]
      },
      {
        name: "Proteins",
        items: [
          { name: "Salmon fillets", amount: "600g", meals: ["Mon Dinner", "Fri Lunch"] },
          { name: "Chicken breast", amount: "800g", meals: ["Tue Dinner", "Sat Lunch"] }
        ]
      },
      {
        name: "Pantry Staples",
        items: [
          { name: "Extra virgin olive oil", amount: "200ml" },
          { name: "Feta cheese", amount: "200g", meals: ["Wed Lunch", "Sun Dinner"] }
        ]
      },
      {
        name: "Herbs & Spices",
        items: [
          { name: "Fresh basil", amount: "1 bunch", meals: ["Multiple"] },
          { name: "Oregano (dried)", amount: "1 tsp" }
        ]
      }
    ],
    total_cost_estimate: "$85-95",
    shopping_time_estimate: "30-40 minutes"
  };
}
```

---

## 🔧 API ARCHITEKTÚRA

### Core API Endpoints

```
Authentication & Users:
POST   /api/v1/auth/signup
POST   /api/v1/auth/login
GET    /api/v1/users/me
PATCH  /api/v1/users/me/profile

Meal Plans:
POST   /api/v1/meal-plans/generate        # Initial 3-day trial
GET    /api/v1/meal-plans/current         # Current week's plan
GET    /api/v1/meal-plans/archive         # Past weeks (Full Access only)
GET    /api/v1/meal-plans/{id}
GET    /api/v1/meal-plans/{id}/pdf

Recipes:
GET    /api/v1/recipes                    # Browse all (Full Access)
GET    /api/v1/recipes/{id}
GET    /api/v1/recipes/seasonal           # Seasonal variations
POST   /api/v1/recipes/{id}/swap          # Meal swap suggestions

Shopping Lists:
GET    /api/v1/shopping-lists/current
GET    /api/v1/shopping-lists/{week_id}
PATCH  /api/v1/shopping-lists/{id}/items/{item_id}/check  # Mark as purchased

Subscriptions:
POST   /api/v1/subscriptions/create
GET    /api/v1/subscriptions/current
POST   /api/v1/subscriptions/cancel
POST   /api/v1/subscriptions/resume
POST   /api/v1/webhooks/stripe            # Stripe webhooks

AI Generation (Internal):
POST   /api/internal/ai/generate-meal-plan
POST   /api/internal/ai/suggest-swaps
POST   /api/internal/ai/generate-shopping-list
```

---

## 🤖 AUTOMATIZÁLT FOLYAMATOK

### 1. Heti Meal Plan Generálás (Cron Jobs)

```yaml
# Kubernetes CronJob
apiVersion: batch/v1
kind: CronJob
metadata:
  name: weekly-meal-plan-generator
spec:
  schedule: "0 6 * * 1"  # Every Monday 6 AM UTC
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: meal-planner
            image: oliveandketo/meal-planner:latest
            env:
            - name: OPENAI_API_KEY
              valueFrom:
                secretKeyRef:
                  name: openai-secret
                  key: api-key
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: db-secret
                  key: url
```

**Generálási Logika:**
```javascript
async function generateUserWeeklyPlan(user) {
  // 1. Prompt építés személyre szabással
  const prompt = `
    Generate a 7-day Mediterranean Keto meal plan for:
    - Age: ${user.profile.age_range}
    - Goal: ${user.profile.goal}
    - Cooking time: ${user.profile.cooking_time}
    - Restrictions: ${user.profile.restrictions.join(', ')}
    - Previous meals (avoid): ${await getRecentMeals(user, 4)}
    - Season: ${getCurrentSeason()}

    Requirements:
    - 21 recipes (3 meals/day × 7 days)
    - Mediterranean ingredients (olive oil, fish, vegetables, herbs)
    - Keto macros: <30g net carbs/day, 70% fat, 25% protein
    - Variety: No meal repeated within 4 weeks
    - Include breakfast, lunch, dinner

    Return JSON format with:
    - Recipe name
    - Ingredients with amounts
    - Step-by-step instructions
    - Prep time, cook time
    - Macros per serving
    - Serving size
  `;

  // 2. OpenAI API call
  const response = await openai.chat.completions.create({
    model: "gpt-4",
    messages: [
      { role: "system", content: "You are a Mediterranean Keto nutritionist and chef." },
      { role: "user", content: prompt }
    ],
    temperature: 0.7,
    response_format: { type: "json_object" }
  });

  // 3. Parse és validálás
  const mealPlan = JSON.parse(response.choices[0].message.content);

  // 4. Nutritional validation
  const validation = await validateNutrition(mealPlan);
  if (!validation.isValid) {
    throw new Error(`Nutrition validation failed: ${validation.errors}`);
  }

  // 5. Generate shopping list
  const shoppingList = await generateShoppingList(mealPlan.meals);

  // 6. Generate PDFs
  const pdf = await generatePDF({
    plan: mealPlan,
    shoppingList: shoppingList,
    user: user
  });

  return {
    meal_plan: mealPlan,
    shopping_list: shoppingList,
    pdf_url: await uploadToS3(pdf)
  };
}
```

---

### 2. Email Automation (Transactional + Marketing)

**Email Service Provider: SendGrid vagy Postmark**

```javascript
// Email Templates
const EMAIL_TEMPLATES = {
  // Onboarding
  'welcome_trial': {
    subject: '🌿 Welcome to Olive & Keto - Your 3-Day Plan is Ready!',
    trigger: 'user.created'
  },

  // Nurture (Trial → Paid)
  'trial_day1': {
    subject: 'Day 1: How are you enjoying your Mediterranean meals?',
    trigger: 'trial.day1'
  },
  'trial_day2': {
    subject: '🔓 Unlock 21 Weekly Recipes (Last Chance!)',
    trigger: 'trial.day2'
  },

  // Full Access
  'full_access_welcome': {
    subject: '🎉 Welcome to Full Access! Your First Week Inside',
    trigger: 'subscription.created'
  },
  'monday_motivation': {
    subject: '🌿 Your Mediterranean Week is Ready! (21 New Recipes)',
    trigger: 'weekly_plan.generated',
    schedule: 'every_monday_8am'
  },

  // Retention
  'subscription_ending': {
    subject: '😢 We'll miss you - Here's 50% off to stay',
    trigger: 'subscription.cancelled',
    delay: '7_days_before_end'
  },

  // Billing
  'payment_success': {
    subject: '✅ Payment Confirmed - Your Access Continues',
    trigger: 'invoice.paid'
  },
  'payment_failed': {
    subject: '⚠️ Payment Issue - Update Your Card',
    trigger: 'invoice.payment_failed'
  }
};

// Email Scheduler
async function scheduleWeeklyEmails() {
  const activeUsers = await db.users.find({
    subscription_status: 'active',
    email_preferences: { weekly_email: true }
  });

  for (const user of activeUsers) {
    await emailQueue.add('send_weekly_email', {
      user_id: user.id,
      template: 'monday_motivation',
      send_at: getNextMonday().setHours(8, 0, 0) // User's timezone
    });
  }
}
```

---

### 3. Meal Swap System (API)

```javascript
POST /api/v1/recipes/{id}/swap
{
  "current_meal_id": "recipe_123",
  "meal_type": "lunch",
  "reason": "dont_like_fish" // vagy "too_complex" vagy "missing_ingredient"
}

RESPONSE:
{
  "alternatives": [
    {
      "id": "recipe_456",
      "name": "Grilled Chicken with Greek Salad",
      "reason": "Similar macros, no fish",
      "macros": {
        "calories": 420,
        "protein": 38,
        "fat": 28,
        "carbs": 6
      },
      "prep_time": "25 min",
      "match_score": 0.92
    },
    {
      "id": "recipe_789",
      "name": "Mediterranean Vegetable Frittata",
      "reason": "Vegetarian alternative",
      "macros": {...},
      "prep_time": "30 min",
      "match_score": 0.85
    }
  ]
}
```

**Swap Algoritmus:**
```javascript
async function suggestMealSwaps(currentMeal, reason, userProfile) {
  // 1. Criteria
  const criteria = {
    meal_type: currentMeal.type, // breakfast/lunch/dinner
    macro_range: {
      calories: [currentMeal.calories * 0.9, currentMeal.calories * 1.1],
      protein: [currentMeal.protein * 0.85, currentMeal.protein * 1.15],
      carbs: [0, 10] // Keto limit
    },
    exclude_ingredients: getExcludedIngredients(reason, userProfile),
    max_prep_time: currentMeal.prep_time * 1.5
  };

  // 2. Query recipe database
  const alternatives = await db.recipes.find({
    type: criteria.meal_type,
    'nutrition.calories': { $gte: criteria.macro_range.calories[0], $lte: criteria.macro_range.calories[1] },
    'nutrition.net_carbs': { $lte: 10 },
    'ingredients.name': { $nin: criteria.exclude_ingredients },
    prep_time_minutes: { $lte: criteria.max_prep_time }
  }).limit(5);

  // 3. Rank by similarity (AI-based)
  const ranked = await rankBySimilarity(currentMeal, alternatives);

  return ranked;
}
```

---

## 💾 DATABASE SCHEMA

```sql
-- Users Table
CREATE TABLE users (
  id UUID PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255),
  name VARCHAR(255),
  profile JSONB, -- {age_range, goal, cooking_time, restrictions}
  subscription_status VARCHAR(50), -- trial, active, cancelled, expired
  subscription_id VARCHAR(255), -- Stripe subscription ID
  subscription_plan VARCHAR(50), -- monthly, annual
  current_period_end TIMESTAMP,
  trial_started_at TIMESTAMP,
  trial_expires_at TIMESTAMP,
  upgraded_at TIMESTAMP,
  cancelled_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Meal Plans Table
CREATE TABLE meal_plans (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  week_start_date DATE,
  meals JSONB, -- Array of 21 meals
  shopping_list JSONB,
  nutritional_summary JSONB,
  pdf_url VARCHAR(500),
  generated_at TIMESTAMP DEFAULT NOW(),
  accessed_at TIMESTAMP
);

-- Recipes Table (Recipe Library)
CREATE TABLE recipes (
  id UUID PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  description TEXT,
  meal_type VARCHAR(50), -- breakfast, lunch, dinner
  ingredients JSONB, -- [{name, amount, unit}]
  instructions JSONB, -- [step1, step2, ...]
  nutrition JSONB, -- {calories, protein, fat, carbs, net_carbs}
  prep_time_minutes INTEGER,
  cook_time_minutes INTEGER,
  servings INTEGER,
  tags JSONB, -- [mediterranean, keto, fish, vegetarian]
  season VARCHAR(50), -- spring, summer, fall, winter, all
  difficulty VARCHAR(50), -- easy, medium, hard
  image_url VARCHAR(500),
  created_at TIMESTAMP DEFAULT NOW()
);

-- User Meal History (for avoiding repetition)
CREATE TABLE user_meal_history (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  recipe_id UUID REFERENCES recipes(id),
  meal_plan_id UUID REFERENCES meal_plans(id),
  served_date DATE,
  meal_type VARCHAR(50),
  rating INTEGER, -- 1-5 stars (optional user feedback)
  created_at TIMESTAMP DEFAULT NOW()
);

-- Subscriptions (Mirror Stripe data)
CREATE TABLE subscriptions (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  stripe_subscription_id VARCHAR(255) UNIQUE,
  stripe_customer_id VARCHAR(255),
  plan VARCHAR(50), -- monthly, annual
  status VARCHAR(50), -- active, cancelled, past_due, unpaid
  current_period_start TIMESTAMP,
  current_period_end TIMESTAMP,
  cancel_at_period_end BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Email Queue (for scheduled emails)
CREATE TABLE email_queue (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  template VARCHAR(100),
  recipient_email VARCHAR(255),
  subject VARCHAR(500),
  data JSONB,
  scheduled_for TIMESTAMP,
  sent_at TIMESTAMP,
  status VARCHAR(50), -- pending, sent, failed
  error_message TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);
```

---

## 🎯 FEATURE GATING (Trial vs Full Access)

```javascript
// Middleware: Check subscription status
async function requireFullAccess(req, res, next) {
  const user = await getUser(req.user_id);

  if (user.subscription_status === 'trial') {
    const trialExpired = new Date() > new Date(user.trial_expires_at);

    if (trialExpired) {
      return res.status(403).json({
        error: 'trial_expired',
        message: 'Your 3-day trial has ended. Upgrade to continue.',
        upgrade_url: '/pricing'
      });
    }
  }

  if (!['active', 'trial'].includes(user.subscription_status)) {
    return res.status(403).json({
      error: 'subscription_required',
      message: 'This feature requires an active subscription.',
      upgrade_url: '/pricing'
    });
  }

  next();
}

// Feature Access Control
const FEATURE_ACCESS = {
  trial: {
    meal_plan_days: 3,
    recipes_access: 'limited', // Only 3-day plan recipes
    pdf_download: true,
    shopping_list: true,
    meal_swaps: false,
    past_plans: false,
    weekly_emails: false,
    seasonal_library: false
  },

  active: {
    meal_plan_days: 7,
    recipes_access: 'full', // All 21 weekly + library
    pdf_download: true,
    shopping_list: true,
    meal_swaps: true,
    past_plans: true,
    weekly_emails: true,
    seasonal_library: true
  }
};

// Usage example
if (!hasAccess(user, 'meal_swaps')) {
  return showUpgradePrompt('Unlock meal swaps with Full Access');
}
```

---

## 📊 METRIKÁK & MONITORING

### Key Metrics to Track

```javascript
// User Lifecycle Metrics
const METRICS = {
  // Acquisition
  'trial_signups_total': 'Counter',
  'trial_signup_source': 'Label', // organic, paid, referral

  // Activation
  'trial_pdf_downloads': 'Counter',
  'trial_shopping_list_views': 'Counter',

  // Conversion
  'trial_to_paid_conversion_rate': 'Percentage',
  'upgrade_within_3days': 'Counter',
  'upgrade_funnel_dropoff': 'Gauge',

  // Revenue
  'mrr': 'Gauge', // Monthly Recurring Revenue
  'arr': 'Gauge', // Annual Recurring Revenue
  'avg_revenue_per_user': 'Gauge',

  // Engagement
  'weekly_plan_open_rate': 'Percentage',
  'monday_email_click_rate': 'Percentage',
  'meal_swaps_per_user': 'Histogram',
  'pdf_downloads_per_week': 'Histogram',

  // Retention
  'churn_rate_monthly': 'Percentage',
  'subscription_lifetime_days': 'Histogram',
  'cancellation_reasons': 'Label',

  // Content
  'meal_plan_generation_time': 'Histogram',
  'ai_generation_success_rate': 'Percentage',
  'recipe_popularity': 'Counter'
};
```

---

## ✅ QUALITY ASSURANCE CHECKLIST

### Biztosítjuk, hogy teljesítjük az ígéreteket:

#### ✅ 21 Új Recept Hetente
- [ ] Cron job minden hétfőn 6:00 AM UTC lefut
- [ ] AI generál 21 egyedi receptet (3 meal × 7 day)
- [ ] Nincs ismétlés 4 héten belül
- [ ] Minden recept validálva (keto macros, mediterrán)
- [ ] Monitoring: Alert ha generation fails

#### ✅ Szervezett Bevásárlólisták
- [ ] Automatikus category grouping (produce, protein, pantry)
- [ ] Ingredient deduplication (összeadja az azonos hozzávalókat)
- [ ] Meal reference (melyik recepthez kell)
- [ ] Estimated cost
- [ ] Printable format

#### ✅ Printable PDFs
- [ ] PDF generálás minden hétfőn
- [ ] Kitchen-ready format (step-by-step, large text)
- [ ] Checkboxes bevásárlólistán
- [ ] Nutrition info minden ételnél
- [ ] S3 storage + CDN delivery

#### ✅ Fresh Meal Plans Every Week
- [ ] Személyre szabott minden usernek
- [ ] Season-aware (tavasz, nyár, ősz, tél)
- [ ] User feedback loop (ratings → future plans)

#### ✅ Meal Swap Suggestions
- [ ] Swap API működik
- [ ] 3-5 alternatíva per meal
- [ ] Macro matching ±10%
- [ ] Ingredient exclusion figyelése

#### ✅ Access to Past Weeks' Plans
- [ ] Archive page minden hétre
- [ ] Unlimited access Full Access users-nek
- [ ] Re-download PDFs

#### ✅ Weekly Monday Motivation Email
- [ ] Automated send minden hétfőn 8 AM (user timezone)
- [ ] Personalized content
- [ ] PDF download link
- [ ] Unsubscribe option

#### ✅ Cancel Anytime
- [ ] One-click cancel from dashboard
- [ ] No retention dark patterns
- [ ] Access until period end
- [ ] Clear refund policy

---

## 🚀 IMPLEMENTATION ROADMAP

### Phase 1: MVP (2-3 weeks)
- [ ] User auth + Stripe integration
- [ ] Manual meal plan generation (admin panel)
- [ ] Basic PDF generation
- [ ] Trial → Paid flow
- [ ] Stripe webhooks handling

### Phase 2: Automation (2-3 weeks)
- [ ] AI meal plan generation (OpenAI)
- [ ] Cron job setup (weekly generation)
- [ ] Email automation (SendGrid)
- [ ] Shopping list algorithm
- [ ] Feature gating middleware

### Phase 3: Polish (1-2 weeks)
- [ ] Meal swap system
- [ ] Past plans archive
- [ ] User preferences refinement
- [ ] PDF template improvements
- [ ] Mobile responsive dashboard

### Phase 4: Scale (Ongoing)
- [ ] Recipe library expansion (200+ recipes)
- [ ] Seasonal variations
- [ ] User ratings & feedback
- [ ] A/B testing email templates
- [ ] Analytics dashboard

---

## 💰 COST ESTIMATION (Monthly)

```
Infrastructure:
- Hosting (AWS/Vercel): $50-100
- Database (PostgreSQL): $25-50
- S3 Storage (PDFs): $10-20
- CDN (CloudFront): $20-30

APIs:
- OpenAI (meal generation): $100-300 (depends on volume)
- SendGrid (emails): $15-50
- Stripe (payment): 2.9% + $0.30 per transaction

Total Fixed Costs: ~$220-550/month

Break-even at ~25-60 paying customers ($8.99/mo)
```

---

## 🔐 SECURITY & COMPLIANCE

- [ ] GDPR compliance (EU users)
- [ ] Data encryption at rest & transit
- [ ] PCI DSS (Stripe handles payments)
- [ ] User data export feature
- [ ] Account deletion (right to be forgotten)
- [ ] Email unsubscribe (CAN-SPAM compliance)

---

## 📞 SUPPORT & CANCELLATION

**Customer Support Channels:**
- Email: support@oliveandketo.com
- FAQ/Help Center
- Priority support for Annual subscribers

**Cancellation Flow:**
1. User clicks "Cancel Subscription" in dashboard
2. Exit survey (1 question: Why leaving?)
3. Offer: 50% discount for 3 months
4. If still cancel: Confirm cancellation
5. Access continues until period end
6. Goodbye email + Win-back campaign after 30 days

---

## 🎉 SUCCESS METRICS (Target)

**Month 1-3:**
- 100 trial signups
- 20% trial → paid conversion (20 paying)
- $180 MRR

**Month 4-6:**
- 500 total users
- 30% trial → paid conversion
- 100 paying customers
- $900 MRR

**Month 7-12:**
- 2000 total users
- 35% conversion
- 500 paying customers
- $4,500 MRR
- 50% on annual plans

---

## 📝 NOTES

**Critical Success Factors:**
1. **Meal quality:** AI must generate actually good, diverse recipes
2. **Reliability:** Weekly emails & PDFs MUST go out on time
3. **Personalization:** Users must feel it's tailored to them
4. **Simplicity:** Onboarding must be <60 seconds
5. **Value perception:** $0.30/meal positioning is key

**Risks to Mitigate:**
- AI generation fails → Fallback to curated recipe database
- High churn → Improve onboarding, add more value
- Low conversion → A/B test pricing, improve trial experience
- Content repetition → Track meal history, expand recipe DB

---

**Prepared by:** Claude
**Date:** 2025-11-09
**Version:** 1.0
