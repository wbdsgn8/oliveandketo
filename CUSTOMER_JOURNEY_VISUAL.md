# 🛤️ Full Access Customer Journey - Vizuális Folyamat

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          DISCOVERY PHASE                                 │
│                                                                          │
│  User lands on website → Fills form (60 sec) → Clicks "Generate Plan"  │
│                                ↓                                         │
│                    ✉️  Email: Welcome + PDF Link                        │
└─────────────────────────────────────────────────────────────────────────┘
                                 ↓
┌─────────────────────────────────────────────────────────────────────────┐
│                        FREE TRIAL (3 DAYS)                               │
│                                                                          │
│  Day 1: ✅ 3-Day Meal Plan + Shopping List + PDF                        │
│  Day 2: 📧 "How's it going?" + Tips                                     │
│  Day 3: 🔓 "Upgrade now - 21 more recipes!" + Urgency                   │
│                                                                          │
│  User Experience:                                                        │
│  ✅ Full access to Day 1-3                                              │
│  🔒 Days 4-7 locked (upgrade prompt)                                    │
│  🔒 Meal swaps locked                                                   │
│  🔒 Past plans locked                                                   │
└─────────────────────────────────────────────────────────────────────────┘
                                 ↓
                          USER DECISION
                                 ↓
                 ┌───────────────┴───────────────┐
                 ↓                               ↓
        🚀 UPGRADES                     ❌ DOESN'T UPGRADE
                 ↓                               ↓
┌────────────────────────────┐      Trial ends → Re-engagement
│   PAYMENT (STRIPE)         │      emails for 30 days
│                            │
│  Monthly: $8.99/mo         │
│  Annual: $49/year          │
└────────────────────────────┘
                 ↓
┌─────────────────────────────────────────────────────────────────────────┐
│                       FULL ACCESS ACTIVATED                              │
│                                                                          │
│  Instant Unlock:                                                         │
│  ✅ 7-Day meal plans (21 recipes/week)                                  │
│  ✅ Organized shopping lists                                            │
│  ✅ Printable PDFs                                                      │
│  ✅ Meal swap system                                                    │
│  ✅ Past weeks archive                                                  │
│  ✅ Weekly Monday emails                                                │
│                                                                          │
│  📧 Email: "Welcome to Full Access!" + First week plan                  │
└─────────────────────────────────────────────────────────────────────────┘
                                 ↓
┌─────────────────────────────────────────────────────────────────────────┐
│                    WEEKLY AUTOMATED CYCLE                                │
│                                                                          │
│  🤖 EVERY MONDAY 6:00 AM (Automated):                                   │
│                                                                          │
│  1. AI generates personalized 21 recipes (7 days × 3 meals)            │
│     ├─ Considers user profile (age, goal, restrictions)                │
│     ├─ Avoids repetition (checks last 4 weeks)                         │
│     ├─ Season-aware (winter → hearty meals)                            │
│     └─ Keto validated (<30g carbs/day, Mediterranean)                  │
│                                                                          │
│  2. Creates organized shopping list                                     │
│     ├─ Grouped by category (Produce, Protein, Pantry)                  │
│     ├─ Deduplicates ingredients                                        │
│     ├─ Shows which meal needs each item                                │
│     └─ Estimates cost                                                   │
│                                                                          │
│  3. Generates kitchen-ready PDF                                         │
│     ├─ Full week meal plan                                             │
│     ├─ Step-by-step recipes                                            │
│     ├─ Shopping list with checkboxes                                   │
│     └─ Nutrition info                                                   │
│                                                                          │
│  4. Sends Monday Motivation Email (8:00 AM user time)                   │
│     ├─ "Your Mediterranean Week is Ready!"                             │
│     ├─ Highlights 3 featured meals                                     │
│     ├─ PDF download link                                               │
│     ├─ Direct link to dashboard                                        │
│     └─ Motivational quote                                              │
│                                                                          │
│  User Actions Available:                                                │
│  📥 Download PDF                                                        │
│  🔄 Swap meals (if don't like fish, allergies, etc)                    │
│  🛒 View shopping list online                                           │
│  📖 Browse past weeks                                                   │
│  ⭐ Rate recipes (improves future plans)                                │
└─────────────────────────────────────────────────────────────────────────┘
                                 ↓
                       CONTINUOUS VALUE LOOP
                                 ↓
┌─────────────────────────────────────────────────────────────────────────┐
│                         RETENTION PHASE                                  │
│                                                                          │
│  Week 1-4:  New user onboarding emails (tips, guides)                   │
│  Week 5+:   Regular Monday meal plans + occasional recipes               │
│  Month 3:   "How are we doing?" feedback survey                         │
│  Month 6:   "Upgrade to Annual?" (save $59/year)                        │
│                                                                          │
│  If user cancels:                                                        │
│  1. Exit survey (1 question)                                            │
│  2. Offer 50% discount (3 months)                                       │
│  3. Access continues until period end                                   │
│  4. Win-back campaign (30 days later)                                   │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 🔄 API FLOW DIAGRAM

```
┌──────────────┐
│   FRONTEND   │
│   (React)    │
└──────┬───────┘
       │
       ↓
┌──────────────────────────────────────────────────────────────────┐
│                        API GATEWAY                                │
│                     (Express.js / Fastify)                        │
└───┬──────────┬──────────┬──────────┬──────────┬─────────────────┘
    │          │          │          │          │
    ↓          ↓          ↓          ↓          ↓
┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐
│ Auth   │ │ Meal   │ │Recipe │ │Shopping│ │Billing │
│Service │ │Planner │ │Service│ │ List   │ │Service │
└───┬────┘ └───┬────┘ └───┬───┘ └───┬────┘ └───┬────┘
    │          │          │          │          │
    ↓          ↓          ↓          ↓          ↓
┌─────────────────────────────────────────────────────────┐
│              PostgreSQL Database                         │
│  Tables: users, meal_plans, recipes, subscriptions      │
└─────────────────────────────────────────────────────────┘

External Services:
├─ 🤖 OpenAI API (meal generation)
├─ 💳 Stripe (payments + webhooks)
├─ 📧 SendGrid (email delivery)
├─ 📦 AWS S3 (PDF storage)
└─ 📊 Analytics (Mixpanel / Amplitude)
```

---

## ⏰ WEEKLY AUTOMATION TIMELINE

```
SUNDAY 11:59 PM
└─ Prepare for Monday generation
   └─ Query all active subscribers
   └─ Check user preferences updates

MONDAY 6:00 AM UTC
└─ 🤖 Start meal plan generation
   ├─ Generate plans for all users (parallel)
   ├─ AI generates 21 recipes per user
   ├─ Validate nutrition (keto macros)
   ├─ Create shopping lists
   └─ Generate PDFs

MONDAY 7:00 AM UTC
└─ 📦 Upload PDFs to S3

MONDAY 8:00 AM (USER LOCAL TIME)
└─ 📧 Send Monday Motivation emails
   └─ Personalized for each user
   └─ Includes PDF link + highlights

MONDAY-SUNDAY
└─ Users access their plans
   ├─ Download PDFs
   ├─ Swap meals if needed
   ├─ Check shopping lists
   └─ Rate recipes (optional)

SUNDAY 11:59 PM
└─ Cycle repeats →
```

---

## 💡 KEY SUCCESS METRICS

```
┌─────────────────────────────────────────────┐
│          METRIC DASHBOARD                    │
├─────────────────────────────────────────────┤
│                                              │
│  📊 Trial → Paid Conversion: 25-35%         │
│  📈 Monthly Churn Rate: <10%                │
│  ⭐ Average Recipe Rating: >4.0/5.0         │
│  📥 PDF Download Rate: >70% of users        │
│  📧 Email Open Rate: >35%                   │
│  🔄 Meal Swap Usage: 2-3 per week          │
│  💰 MRR Growth: +15% month-over-month       │
│  🎯 Customer LTV: $80-120                   │
│                                              │
└─────────────────────────────────────────────┘
```

---

## 🎯 VALUE PROPOSITION (User's Perspective)

```
PROBLEM:
❌ Meal planning is time-consuming (2-3 hours/week)
❌ Don't know which foods are keto + Mediterranean
❌ Grocery shopping is chaotic
❌ Recipes are too complex or boring
❌ Calorie/macro tracking is tedious

     ↓

SOLUTION (Full Access):
✅ 21 NEW recipes every Monday (0 planning time)
✅ All meals validated as Keto + Mediterranean
✅ Organized shopping list (30 min grocery trip)
✅ Simple, delicious recipes (Mediterranean flavors)
✅ Macros calculated automatically

     ↓

RESULT:
🎉 Save 2-3 hours/week
🎉 Lose weight sustainably
🎉 Enjoy restaurant-quality meals at home
🎉 Never run out of ideas
🎉 Cost: $0.30/meal ($8.99 for 30 meals)
```

---

## 🚨 RISK MITIGATION

| Risk | Impact | Mitigation |
|------|--------|------------|
| AI generation fails | High | Fallback to curated recipe DB |
| Email delivery fails | Medium | Monitor SendGrid, retry queue |
| Stripe payment fails | High | Auto-retry 3x, email user |
| High churn rate | High | Exit surveys, improve value |
| Content repetition | Medium | Track 4-week history, larger DB |
| Slow PDF generation | Low | Pre-generate on Sunday night |
| OpenAI cost spike | Medium | Implement caching, rate limits |
| GDPR compliance | High | User data export, right to delete |

---

## 📦 TECH STACK RECOMMENDATION

```
Frontend:
├─ React / Next.js
├─ TailwindCSS
└─ Stripe Elements (payment UI)

Backend:
├─ Node.js + Express/Fastify
├─ PostgreSQL (Supabase or Railway)
├─ Redis (caching)
└─ Bull Queue (job scheduling)

AI & Automation:
├─ OpenAI GPT-4 (meal generation)
├─ Puppeteer (PDF generation)
└─ Node-cron (scheduling)

External Services:
├─ Stripe (payments)
├─ SendGrid (emails)
├─ AWS S3 + CloudFront (storage + CDN)
├─ Vercel / Railway (hosting)
└─ Sentry (error tracking)

DevOps:
├─ GitHub Actions (CI/CD)
├─ Docker (containerization)
└─ Kubernetes (optional, for scale)
```

---

## ✅ MVP LAUNCH CHECKLIST

**Must-Have (Launch Day):**
- [ ] User signup + login
- [ ] Free 3-day trial generation
- [ ] Stripe payment integration
- [ ] Basic PDF generation
- [ ] Welcome email automation
- [ ] Trial → Paid upgrade flow
- [ ] Cancel subscription
- [ ] Basic dashboard (view current plan)

**Nice-to-Have (Week 2-4):**
- [ ] Weekly automated meal generation
- [ ] Monday motivation emails
- [ ] Meal swap system
- [ ] Past plans archive
- [ ] Shopping list optimization
- [ ] User preferences editing
- [ ] Recipe ratings

**Future (Month 2+):**
- [ ] Seasonal variations library
- [ ] Recipe favorites
- [ ] Meal prep guides
- [ ] Mobile app (iOS/Android)
- [ ] Referral program
- [ ] Annual plan incentives

---

**Summary:** This system delivers on all promises automatically. Users get fresh, personalized meal plans every Monday without lifting a finger. The key is robust automation + quality AI generation + reliable delivery.
