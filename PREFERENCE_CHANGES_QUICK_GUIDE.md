# ⚡ Gyors Útmutató: Változások Kezelése (Nulla Emberi Beavatkozással)

## 🎯 A 7 Leggyakoribb Változás és Kezelésük

### 1️⃣ "Mégsem szeretem a brokkolit" 🥦

**Felhasználói Akció:**
```
Recipe Page → Broccoli ingredient → 🚫 Block Button
```

**Automatikus Folyamat:**
```
1. Azonnal hozzáadás: disliked_ingredients.push('broccoli')
2. Ellenőrzés: Van broccoli a heti tervben?
   ├─ Igen → Auto-swap azokat az ételeket
   └─ Nem → OK
3. Email: "Broccoli blocked. We swapped 2 meals this week."
4. Jövő hétfőtől: AI automatikusan kihagyja a brokkolit
```

**Implementáció:**
```javascript
POST /api/v1/preferences/block-ingredient
{ "ingredient": "broccoli" }

→ Database update (instant)
→ Current week check & swap (if needed)
→ Email notification
→ Future plans auto-exclude
```

---

### 2️⃣ "Cél váltás: Fogyás → Izomépítés" 💪

**Felhasználói Akció:**
```
Settings → Goal → Select "Muscle Gain" → Save
```

**Automatikus Folyamat:**
```
1. Makro újraszámítás:
   Régi: 1,600 cal, 120g protein, 106g fat, 25g carbs
   Új:  2,400 cal, 150g protein, 140g fat, 30g carbs

2. Ellenőrzés: Elkezdődött a hét?
   ├─ NEM (pl. vasárnap este)
   │   └─ Azonnali regenerálás új makrókkal
   └─ IGEN (pl. szerda)
       └─ Következő hétfőn lép életbe

3. UI Notification:
   "✅ Goal updated! New macros take effect Monday Nov 11."
```

**Implementáció:**
```javascript
PATCH /api/v1/users/me/profile
{ "goal": "muscle_gain" }

→ Recalculate macros
→ Check if week started
→ Regenerate OR schedule for next Monday
→ Show before/after macro comparison
```

---

### 3️⃣ "Túl sok hal, meguntam" 🐟

**Felhasználói Út:**
```
Felhasználó 3x swapol hal ételt:
  Week 1: Swaps Salmon → Chicken
  Week 2: Swaps Tuna → Beef
  Week 3: Swaps Cod → Vegetarian
```

**AI Észreveszi (3. swap után):**
```
Pattern detected: 3 fish swaps in 3 weeks

Automatikus Változtatás:
  meal_frequency.fish_per_week: "3-4" → "0-1"

Email User:
  "We noticed you've swapped fish 3 times.
   We've reduced fish to 1 meal/week.
   Change this anytime in Settings."
```

**Implementáció:**
```javascript
POST /api/v1/meals/swap
{ "meal_id": "...", "reason": "dont_like_fish" }

→ Log swap with reason
→ Count fish swaps (3+ in 4 weeks?)
→ Auto-adjust fish frequency
→ Notify user (transparent)
→ User can override in Settings
```

---

### 4️⃣ "Nincs időm 45 perces receptekhez" ⏱️

**Felhasználói Viselkedés:**
```
User swaps 4 meals with reason: "too_complex"
  - Monday: 45min recipe → 25min recipe
  - Wednesday: 50min → 30min
  - Friday: 40min → 20min
  - Next week: 45min → 25min
```

**AI Reakció:**
```
Pattern: 4 swaps for "too_complex" reason

Auto-Adjustment:
  cooking_time: "45_min" → "30_min"

Future Meal Plans:
  - Max prep time: 30 minutes
  - Prefer "Quick & Easy" recipes
  - Filter out complex recipes
```

**Implementáció:**
```javascript
if (complexSwaps >= 4 && user.cooking_time !== '15_min') {
  await db.users.update({
    'preferences.cooking_time': '30_min'
  });

  sendEmail({
    template: 'cooking_time_adjusted',
    message: "We've adjusted your meal plans to quicker recipes (max 30 min)."
  });
}
```

---

### 5️⃣ "Allergia felfedezés: Garnélarák" 🦐

**Felhasználói Akció:**
```
Settings → Allergies → Add "Shellfish" → Save
```

**Azonnali Hatás:**
```
1. Add to restrictions.allergies: ['shellfish']

2. Scan current week:
   Found 1 meal with shrimp (Gambas al Ajillo)
   → Auto-swap to safe alternative (Grilled Chicken)

3. Critical Alert (safety):
   "⚠️ Shellfish allergy added. We've removed shrimp from your plan.
    Future meals will NEVER include shellfish."

4. Future weeks:
   AI prompt: "CRITICAL: User has shellfish allergy. Never include shrimp, crab, lobster, etc."
```

**Implementáció:**
```javascript
POST /api/v1/preferences/add-allergy
{ "allergy": "shellfish" }

→ Add to allergies (CRITICAL flag)
→ Scan & swap current week immediately
→ Red alert notification
→ Future plans: Hard exclude (AI prompt priority)
→ Double validation on generation
```

---

### 6️⃣ "Unalmas lett, mindig ugyanaz" 😴

**AI Detection (Automatikus):**
```
Weekly Analysis (Before generating new plan):

Ingredient Frequency (Last 4 weeks):
  - Chicken: 12 times ❌ (overused)
  - Olive oil: 28 times ✅ (staple, OK)
  - Feta cheese: 9 times ❌ (overused)
  - Salmon: 8 times ❌ (borderline)

Recipe Repetition:
  - "Greek Salad": 3 times ❌
  - "Grilled Chicken": 4 times ❌

Boredom Score: 7/10 → HIGH
```

**AI Auto-Correction:**
```
This Week's Plan Adjustments:
  ✅ Introduce NEW proteins: lamb, duck, sardines
  ✅ New cuisines: Moroccan tagine, Spanish paella
  ✅ Zero chicken this week (give it a break)
  ✅ Zero repeated recipes from past 4 weeks
  ✅ 2 "exotic" Mediterranean dishes

Email Subject: "🌟 Fresh Flavors This Week!"
Body: "We noticed you've been enjoying similar meals.
       This week: Moroccan Lamb, Spanish Gambas, Turkish Kebabs!"
```

**Implementáció:**
```javascript
// Runs before every weekly generation
const boredom = await detectBoredom(user_id);

if (boredom.score > 6) {
  const varietyPrompt = `
    HIGH VARIETY MODE:
    - Avoid: ${boredom.overused_ingredients}
    - Try: ${getSeasonalIngredients()}
    - New cuisines: Moroccan, Turkish, Spanish
    - Exciting techniques: braising, poaching, raw
  `;

  await generateWithVariety(user_id, varietyPrompt);
}
```

---

### 7️⃣ "Vegetáriánus szeretnék lenni" 🥗

**Felhasználói Akció:**
```
Settings → Dietary Type → Select "Vegetarian" → Save
```

**Automatikus Változtatás:**
```
1. Update: dietary_type: "vegetarian"

2. Adjust Frequencies:
   fish_per_week: "3-4" → "0"
   red_meat_per_week: "0-1" → "0"
   vegetarian_meals_per_week: "2-3" → "7"

3. Macro Adjustment:
   Protein sources switch:
   - FROM: Fish, chicken, beef
   - TO: Eggs, cheese, legumes, tofu, tempeh

4. Current Week:
   IF week not started:
     → Regenerate with vegetarian recipes
   ELSE:
     → Offer: "Want to swap meat meals this week? [Yes] [Wait till Monday]"

5. Future Weeks:
   All meals 100% vegetarian Mediterranean
```

**Implementáció:**
```javascript
PATCH /api/v1/preferences/dietary-type
{ "dietary_type": "vegetarian" }

→ Update profile
→ Auto-adjust meal frequencies (0 meat/fish)
→ Recalculate protein sources
→ Offer current week regeneration
→ AI prompt: "100% vegetarian recipes only"
→ Validate: No meat/fish in generated plans
```

---

## 🔄 Preference Change Matrix

| Változás | Hatás Ideje | Auto-Swap Current Week? | Email Notification? | AI Learning? |
|----------|-------------|------------------------|---------------------|--------------|
| Block ingredient | Instant | ✅ Yes (if ingredient present) | ✅ Yes | ❌ No (explicit) |
| Change goal | Next Monday | ⚠️ Only if week not started | ✅ Yes | ❌ No (explicit) |
| Swap meal (3+ times) | Next Monday | ❌ No | ✅ Yes (pattern detected) | ✅ Yes (implicit) |
| Add allergy | INSTANT | ✅ YES (safety critical) | ✅ YES (red alert) | ❌ No (explicit) |
| Change cooking time | Next Monday | ❌ No | ⚠️ Only if auto-adjusted | ⚠️ Yes (if detected) |
| Dietary type change | Next Monday | ⚠️ User chooses | ✅ Yes | ❌ No (explicit) |
| Low engagement | Next Monday | ❌ No | ✅ Re-engagement email | ✅ Yes (churn risk) |

---

## 🎛️ User Control Panel (What They See)

```
┌─────────────────────────────────────────────┐
│         MY PREFERENCES                       │
├─────────────────────────────────────────────┤
│                                              │
│  Current Goal: Muscle Gain  [Edit]          │
│  Changed: Nov 9 (takes effect Nov 11)       │
│                                              │
│  Foods I Avoid:                              │
│  🚫 Broccoli       [Remove]                  │
│  🚫 Shellfish (allergy) [Remove]             │
│  + Add more                                  │
│                                              │
│  Meal Preferences:                           │
│  Fish: ▓▓▓░░░░ 3-4/week  [Auto-adjusted]    │
│  Vegetarian: ▓▓░░░░░ 2/week                  │
│  Cooking Time: ≤30 min                       │
│                                              │
│  What AI Learned:                            │
│  💡 You prefer Italian cuisine               │
│  💡 You swap complex recipes → Shorter now   │
│  💡 High engagement (great job!)             │
│                                              │
│  [Reset All Preferences]                     │
│  [Export My Data]                            │
└─────────────────────────────────────────────┘
```

---

## 🚨 Edge Cases & Safety

### "User blocks everything, no meals possible"

```javascript
if (user.disliked_ingredients.length > 20) {
  return {
    warning: "You've blocked 20+ ingredients. We might struggle to create variety.",
    suggestion: "Consider using Meal Swaps instead of blocking ingredients."
  };
}

// Before generation, validate feasibility
const feasible = await canGenerateMeals(user.preferences);
if (!feasible) {
  sendEmail({
    subject: "⚠️ Your preferences are too restrictive",
    message: "Please review your blocked ingredients. We need at least 30 ingredients to create varied meal plans."
  });
}
```

### "AI ignores preferences (BUG)"

```javascript
// Post-generation validation
async function validateGeneratedPlan(plan, user_preferences) {
  const violations = [];

  plan.meals.forEach(meal => {
    // Check blocked ingredients
    meal.ingredients.forEach(ing => {
      if (user_preferences.disliked_ingredients.includes(ing.name)) {
        violations.push({
          type: 'blocked_ingredient',
          meal: meal.name,
          ingredient: ing.name
        });
      }
    });

    // Check allergies (CRITICAL)
    if (containsAllergen(meal, user_preferences.allergies)) {
      violations.push({
        type: 'ALLERGY_VIOLATION', // Critical
        meal: meal.name
      });
    }
  });

  if (violations.length > 0) {
    // Log for debugging
    await logError('preference_violation', { plan, violations });

    // Auto-fix
    return await regeneratePlan(user, { strict_mode: true });
  }

  return { valid: true };
}
```

---

## 📊 Metrics to Track

```javascript
{
  // Explicit Preference Changes
  "ingredient_blocks_per_week": 450,
  "goal_changes_per_week": 80,
  "allergy_additions_per_week": 5,

  // Implicit Learning (AI)
  "auto_adjustments_per_week": 120,
  "users_accepted_auto_adjustments": "85%",

  // Engagement Impact
  "churn_rate_with_custom_prefs": "5%",
  "churn_rate_default_prefs": "15%",
  // → Custom preferences reduce churn by 67%!

  // Satisfaction
  "avg_plan_rating_custom_prefs": 4.6,
  "avg_plan_rating_default": 3.8,

  // AI Accuracy
  "preference_violations_per_1000_meals": 2, // Very low
  "allergy_violations": 0 // MUST be zero
}
```

---

## ✅ Summary: How It Works

```
┌─────────────────────────────────────────────────┐
│  USER CHANGES SOMETHING                          │
│  (block ingredient, change goal, swap meal)      │
└────────────────┬────────────────────────────────┘
                 ↓
┌─────────────────────────────────────────────────┐
│  SYSTEM DETECTS CHANGE                           │
│  - Explicit: User clicked button                 │
│  - Implicit: AI detected pattern (3+ swaps)      │
└────────────────┬────────────────────────────────┘
                 ↓
┌─────────────────────────────────────────────────┐
│  IMMEDIATE ACTIONS                               │
│  1. Update database (preferences table)          │
│  2. Check current week (needs swap?)             │
│  3. Send notification email (transparency)       │
└────────────────┬────────────────────────────────┘
                 ↓
┌─────────────────────────────────────────────────┐
│  NEXT MEAL GENERATION (Monday 6 AM)              │
│  1. AI reads updated preferences                 │
│  2. Generates plan respecting ALL constraints    │
│  3. Validates (double-check allergies)           │
│  4. Sends plan to user                           │
└─────────────────────────────────────────────────┘
                 ↓
         USER IS HAPPY ✅
    (No human intervention needed)
```

---

**Végső Válasz:** Minden változás önkiszolgáló. User kattint → System azonnal reagál → AI alkalmazkodik → Nulla emberi munka. 🚀
