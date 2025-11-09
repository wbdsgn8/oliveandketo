# 🎛️ Önkiszolgáló Preferencia Kezelő Rendszer

## 🎯 Probléma

Felhasználók változtatnak:
- ❌ "Mégsem szeretem a brokkolit"
- ❌ "Most már izomépítés a célom, nem fogyás"
- ❌ "Rájöttem, hogy allergiás vagyok a garnélarákra"
- ❌ "Túl sok hal, untam meg"
- ❌ "Nincs időm 45 perces receptekhez"
- ❌ "Vegetáriánus szeretnék lenni"

**Követelmény:** Nulla emberi beavatkozás, 100% automatizált.

---

## 🛠️ Megoldás: Multi-Layer Preference System

### 1️⃣ EXPLICIT PREFERENCES (User Settings)

**Dashboard → My Preferences Panel**

```javascript
// User preferences adatmodell
{
  user_id: "usr_123",

  // CORE PROFILE (módosítható bármikor)
  profile: {
    age_range: "30-40",
    goal: "muscle_gain", // weight_loss, maintain, muscle_gain
    activity_level: "moderate", // sedentary, light, moderate, active, very_active
    cooking_time: "30_min", // 15_min, 30_min, 45_min, 60_min
    budget_per_meal: "medium" // low, medium, high
  },

  // DIETARY RESTRICTIONS (add/remove)
  restrictions: {
    allergies: ["shellfish", "tree_nuts"],
    intolerances: ["lactose", "gluten"],
    dietary_type: "pescatarian", // omnivore, vegetarian, pescatarian, vegan
    religious: [] // halal, kosher
  },

  // INGREDIENT DISLIKES (user manually blocks)
  disliked_ingredients: [
    { name: "broccoli", blocked_at: "2025-11-09", reason: "dont_like_taste" },
    { name: "anchovies", blocked_at: "2025-11-05", reason: "too_salty" }
  ],

  // MEAL FREQUENCY PREFERENCES
  meal_frequency: {
    fish_per_week: "3-4", // 0-1, 2-3, 3-4, 5-7
    red_meat_per_week: "0-1",
    vegetarian_meals_per_week: "2-3",
    eggs_per_week: "unlimited"
  },

  // CUISINE PREFERENCES
  cuisine_preferences: {
    greek: true,
    italian: true,
    spanish: true,
    turkish: true,
    moroccan: false // "too exotic for me"
  },

  // COOKING COMPLEXITY
  complexity_preference: "easy", // easy, medium, advanced

  updated_at: "2025-11-09T10:30:00Z"
}
```

**UI Components:**

```jsx
// Dashboard: Preferences Page
<PreferencesPanel>
  <Section title="My Goals">
    <Select
      value={profile.goal}
      onChange={handleGoalChange}
      options={[
        "Weight Loss",
        "Maintain Weight",
        "Muscle Gain"
      ]}
    />
    <Info>Your macros will adjust automatically next Monday</Info>
  </Section>

  <Section title="Dietary Restrictions">
    <Checkbox label="Vegetarian" />
    <Checkbox label="Pescatarian" />
    <Checkbox label="Dairy-Free" />
    <MultiSelect
      label="Allergies"
      options={COMMON_ALLERGENS}
      selected={restrictions.allergies}
    />
  </Section>

  <Section title="Foods I Don't Like">
    <IngredientBlockList>
      {disliked_ingredients.map(item => (
        <BlockedItem key={item.name}>
          {item.name}
          <RemoveButton onClick={() => unblock(item.name)} />
        </BlockedItem>
      ))}
    </IngredientBlockList>
    <AddButton onClick={openIngredientSearch}>
      + Block an ingredient
    </AddButton>
  </Section>

  <Section title="Meal Frequency">
    <Slider
      label="Fish meals per week"
      min={0} max={7}
      value={meal_frequency.fish_per_week}
    />
    <Slider
      label="Vegetarian meals per week"
      min={0} max={7}
    />
  </Section>

  <Section title="Cooking Time">
    <RadioGroup>
      <Radio value="15_min">Quick meals (15 min)</Radio>
      <Radio value="30_min">Normal (30 min)</Radio>
      <Radio value="45_min">Relaxed (45 min)</Radio>
    </RadioGroup>
  </Section>

  <SaveButton onClick={savePreferences}>
    Save Changes (Takes effect next Monday)
  </SaveButton>
</PreferencesPanel>
```

---

### 2️⃣ IMPLICIT PREFERENCES (AI Learning)

**System learns from user behavior WITHOUT user input:**

```javascript
// User behavior tracking
const IMPLICIT_SIGNALS = {

  // Recipe Ratings (when user rates meals)
  recipe_ratings: [
    { recipe_id: "rec_123", rating: 5, rated_at: "2025-11-09" },
    { recipe_id: "rec_456", rating: 2, rated_at: "2025-11-08" },
    { recipe_id: "rec_789", rating: 4, rated_at: "2025-11-07" }
  ],

  // Meal Swaps (what user swaps = dislike signal)
  meal_swaps: [
    {
      original_recipe: "rec_salmon_45min",
      swapped_to: "rec_chicken_25min",
      reason: "too_complex", // or "missing_ingredient" or "dont_like"
      swapped_at: "2025-11-08"
    }
  ],

  // PDF Downloads (engagement signal)
  pdf_downloads: [
    { week: "2025-W45", downloaded: true, download_count: 2 },
    { week: "2025-W44", downloaded: false, download_count: 0 } // Low engagement?
  ],

  // Email Opens & Clicks
  email_engagement: {
    monday_emails_opened: 8,
    monday_emails_sent: 10,
    open_rate: 0.8, // Good engagement
    recipe_clicks: ["rec_123", "rec_456", "rec_789"] // Which recipes they clicked
  },

  // Recipe View Time (how long they looked at a recipe)
  recipe_views: [
    { recipe_id: "rec_123", view_duration_seconds: 180 }, // Read thoroughly
    { recipe_id: "rec_456", view_duration_seconds: 5 }   // Skipped
  ]
};

// AI Analysis
async function analyzeUserBehavior(user_id) {
  const signals = await getImplicitSignals(user_id);

  // Detect patterns
  const insights = {

    // Low-rated recipes → Extract common ingredients to avoid
    disliked_ingredients: await extractFromLowRatings(signals.recipe_ratings),
    // Example: User rated 3 fish recipes as 2/5 → Maybe dislikes fish?

    // Swap patterns → Complexity preference
    prefers_quick_meals: signals.meal_swaps.filter(s => s.reason === 'too_complex').length > 3,

    // Cuisine preference from high ratings
    favorite_cuisines: await extractCuisinePreference(signals.recipe_ratings),

    // Engagement level
    at_risk_churn: signals.email_engagement.open_rate < 0.3 ||
                   signals.pdf_downloads.filter(d => !d.downloaded).length > 2
  };

  return insights;
}

// Apply insights to next meal plan
async function generateSmartMealPlan(user) {
  const explicit = user.preferences; // User manually set
  const implicit = await analyzeUserBehavior(user.id); // AI learned

  const constraints = {
    // Explicit overrides implicit
    blocked_ingredients: [
      ...explicit.disliked_ingredients.map(i => i.name),
      ...implicit.disliked_ingredients // AI detected
    ],

    max_prep_time: explicit.cooking_time === '15_min'
      ? 15
      : implicit.prefers_quick_meals
        ? 25 // AI detected they swap complex meals
        : 45,

    cuisine_weights: {
      greek: explicit.cuisine_preferences.greek ? 0.3 : 0.1,
      italian: explicit.cuisine_preferences.italian ? 0.3 : 0.1,
      // AI boost: If user rated Italian recipes highly
      italian: implicit.favorite_cuisines.includes('italian') ? 0.4 : 0.1
    }
  };

  return await aiMealPlanner.generate(user, constraints);
}
```

---

### 3️⃣ CONTEXTUAL QUICK ACTIONS

**User sees a recipe and says "I don't like broccoli"**

**Option A: In-Recipe Block Button**

```jsx
// Recipe Page UI
<RecipeCard recipe={recipe}>
  <RecipeImage src={recipe.image} />
  <RecipeTitle>{recipe.name}</RecipeTitle>

  <IngredientsList>
    {recipe.ingredients.map(ingredient => (
      <IngredientRow key={ingredient.name}>
        <span>{ingredient.amount} {ingredient.name}</span>
        <BlockButton
          onClick={() => blockIngredient(ingredient.name)}
          tooltip="Never show me recipes with this ingredient"
        >
          🚫 Block
        </BlockButton>
      </IngredientRow>
    ))}
  </IngredientsList>

  <SwapMealButton onClick={handleSwap}>
    🔄 Don't like this meal? Swap it
  </SwapMealButton>
</RecipeCard>
```

**Backend: Instant Block**

```javascript
POST /api/v1/preferences/block-ingredient
{
  "ingredient": "broccoli",
  "reason": "dont_like_taste" // optional
}

// Immediate effect:
async function blockIngredient(user_id, ingredient) {
  // 1. Add to user preferences
  await db.users.update(user_id, {
    disliked_ingredients: arrayAppend(ingredient)
  });

  // 2. Remove from current week (if not started cooking)
  const currentWeek = await getCurrentWeekPlan(user_id);
  const hasBroccoli = currentWeek.meals.filter(m =>
    m.ingredients.some(i => i.name.includes('broccoli'))
  );

  if (hasBroccoli.length > 0) {
    // Auto-swap those meals
    for (const meal of hasBroccoli) {
      const alternative = await suggestSwap(meal, {
        exclude: ['broccoli']
      });
      await swapMeal(user_id, meal.id, alternative.id);
    }

    // Notify user
    await sendEmail({
      template: 'ingredient_blocked_notification',
      to: user.email,
      data: {
        ingredient: 'broccoli',
        swapped_meals_count: hasBroccoli.length,
        new_plan_url: getDashboardUrl(user_id)
      }
    });
  }

  // 3. Future weeks automatically exclude broccoli
  return {
    success: true,
    message: `Broccoli blocked. We've swapped ${hasBroccoli.length} meals this week.`,
    next_week_note: "Future meal plans will avoid broccoli automatically."
  };
}
```

---

### 4️⃣ SMART SWAP SYSTEM WITH MEMORY

**User swaps a meal → System learns WHY**

```jsx
// Swap Modal UI
<SwapMealModal meal={currentMeal}>
  <Title>Why do you want to swap this meal?</Title>

  <ReasonSelector>
    <Reason onClick={() => swap('dont_like_fish')}>
      🐟 Don't like fish
    </Reason>
    <Reason onClick={() => swap('too_complex')}>
      ⏱️ Takes too long to cook
    </Reason>
    <Reason onClick={() => swap('missing_ingredient')}>
      🛒 Missing ingredient
    </Reason>
    <Reason onClick={() => swap('had_recently')}>
      🔁 Had this recently
    </Reason>
    <Reason onClick={() => swap('other')}>
      💬 Other reason (tell us)
    </Reason>
  </ReasonSelector>

  <Alternatives>
    {alternatives.map(alt => (
      <AlternativeCard
        recipe={alt}
        matchScore={alt.match_score}
        onClick={() => confirmSwap(alt)}
      />
    ))}
  </Alternatives>
</SwapMealModal>
```

**Backend: Learn from Swap Reason**

```javascript
POST /api/v1/meals/swap
{
  "meal_id": "meal_123",
  "reason": "dont_like_fish",
  "swap_to": "meal_456" // optional, auto-suggest if empty
}

async function handleMealSwap(user_id, meal_id, reason, swap_to) {
  // 1. Log the swap
  await db.meal_swaps.create({
    user_id,
    original_meal_id: meal_id,
    swapped_to_meal_id: swap_to,
    reason,
    swapped_at: new Date()
  });

  // 2. Extract learnings
  const meal = await db.meals.findById(meal_id);

  if (reason === 'dont_like_fish') {
    // Check if this is a pattern (3+ fish swaps)
    const fishSwaps = await db.meal_swaps.count({
      user_id,
      reason: 'dont_like_fish'
    });

    if (fishSwaps >= 3) {
      // Auto-adjust fish frequency preference
      await db.users.update(user_id, {
        'preferences.meal_frequency.fish_per_week': '0-1' // Reduce fish
      });

      // Notify user
      await sendEmail({
        template: 'preference_auto_adjusted',
        data: {
          message: "We noticed you've swapped fish meals 3 times. We've reduced fish frequency in your future plans. You can adjust this in Settings."
        }
      });
    }
  }

  if (reason === 'too_complex') {
    // Reduce cooking time preference
    const complexSwaps = await db.meal_swaps.count({
      user_id,
      reason: 'too_complex'
    });

    if (complexSwaps >= 3 && user.preferences.cooking_time !== '15_min') {
      await db.users.update(user_id, {
        'preferences.cooking_time': '30_min' // Shorten
      });
    }
  }

  // 3. Perform the swap
  const alternative = swap_to || await suggestBestAlternative(meal, user);
  await replaceMealInPlan(user_id, meal_id, alternative);

  return {
    success: true,
    new_meal: alternative,
    learning_applied: fishSwaps >= 3 ? 'Reduced fish frequency' : null
  };
}
```

---

### 5️⃣ MACRO ADJUSTMENTS (Goal Changes)

**User changes from "Weight Loss" to "Muscle Gain"**

```javascript
// User clicks: Settings → Goal → Muscle Gain

PATCH /api/v1/users/me/profile
{
  "goal": "muscle_gain"
}

async function updateUserGoal(user_id, new_goal) {
  // 1. Update profile
  await db.users.update(user_id, {
    'profile.goal': new_goal,
    'profile.goal_changed_at': new Date()
  });

  // 2. Recalculate macros
  const user = await db.users.findById(user_id);
  const newMacros = calculateMacros(user);

  // MACRO CALCULATIONS
  function calculateMacros(user) {
    const { age_range, goal, activity_level } = user.profile;

    // Estimate TDEE (Total Daily Energy Expenditure)
    const baseTDEE = {
      '18-25': 2200,
      '26-35': 2100,
      '36-45': 2000,
      '46-55': 1900,
      '56-65': 1800,
      '65+': 1700
    }[age_range];

    const activityMultiplier = {
      sedentary: 1.2,
      light: 1.375,
      moderate: 1.55,
      active: 1.725,
      very_active: 1.9
    }[activity_level];

    const tdee = baseTDEE * activityMultiplier;

    // Adjust for goal
    let calories, protein, fat, carbs;

    if (goal === 'weight_loss') {
      calories = tdee - 500; // 500 cal deficit
      protein = 1.6; // g per kg bodyweight (estimate 75kg → 120g)
      fat = 0.75; // 75% of calories from fat (keto)
      carbs = 25; // <25g net carbs (keto)
    }
    else if (goal === 'muscle_gain') {
      calories = tdee + 300; // 300 cal surplus
      protein = 2.0; // Higher protein (estimate 75kg → 150g)
      fat = 0.65; // 65% fat
      carbs = 30; // Slightly more carbs for energy
    }
    else { // maintain
      calories = tdee;
      protein = 1.4;
      fat = 0.70;
      carbs = 25;
    }

    return {
      daily_calories: Math.round(calories),
      daily_protein_g: Math.round(protein * 75), // Estimate 75kg bodyweight
      daily_fat_g: Math.round((calories * fat / 100) / 9), // 9 cal per g fat
      daily_carbs_g: carbs,
      per_meal: {
        calories: Math.round(calories / 3),
        protein: Math.round((protein * 75) / 3),
        fat: Math.round(((calories * fat / 100) / 9) / 3),
        carbs: Math.round(carbs / 3)
      }
    };
  }

  const macros = newMacros;

  // 3. Save new macros
  await db.users.update(user_id, {
    'profile.macros': macros
  });

  // 4. Check if current week needs regeneration
  const currentWeek = await getCurrentWeekPlan(user_id);
  const weekStarted = new Date() > new Date(currentWeek.week_start_date);

  if (!weekStarted) {
    // Week hasn't started → Regenerate immediately
    await regenerateWeekPlan(user_id, macros);

    return {
      success: true,
      message: "Goal updated! This week's meal plan has been regenerated with your new macros.",
      new_macros: macros,
      regenerated: true
    };
  } else {
    // Week already started → Apply next Monday
    return {
      success: true,
      message: "Goal updated! Your new macros will take effect next Monday.",
      new_macros: macros,
      takes_effect: getNextMonday(),
      regenerated: false
    };
  }
}
```

**UI Notification:**

```jsx
<SuccessNotification>
  ✅ Goal updated to Muscle Gain!

  <MacroComparison>
    <Before>
      <h4>Previous (Weight Loss)</h4>
      <p>Calories: 1,600/day</p>
      <p>Protein: 120g</p>
      <p>Fat: 106g</p>
      <p>Carbs: 25g</p>
    </Before>

    <After>
      <h4>New (Muscle Gain)</h4>
      <p>Calories: 2,400/day ↑</p>
      <p>Protein: 150g ↑</p>
      <p>Fat: 140g ↑</p>
      <p>Carbs: 30g ↑</p>
    </After>
  </MacroComparison>

  <EffectiveDate>
    Takes effect: Next Monday (Nov 11)
  </EffectiveDate>
</SuccessNotification>
```

---

### 6️⃣ BOREDOM DETECTION & AUTO-VARIETY

**AI detects user is getting bored (same ingredients too often)**

```javascript
// Weekly check: Before generating new plan
async function detectBoredom(user_id) {
  // Get last 4 weeks of meals
  const recentMeals = await db.meal_plans.find({
    user_id,
    week_start_date: { $gte: getWeeksAgo(4) }
  });

  // Extract ingredient frequency
  const ingredientCounts = {};
  recentMeals.forEach(week => {
    week.meals.forEach(meal => {
      meal.ingredients.forEach(ing => {
        ingredientCounts[ing.name] = (ingredientCounts[ing.name] || 0) + 1;
      });
    });
  });

  // Detect overused ingredients
  const overused = Object.entries(ingredientCounts)
    .filter(([ing, count]) => count > 8) // More than 8 times in 4 weeks
    .map(([ing, count]) => ing);

  // Detect recipe repetition
  const recipeCounts = {};
  recentMeals.forEach(week => {
    week.meals.forEach(meal => {
      recipeCounts[meal.recipe_id] = (recipeCounts[meal.recipe_id] || 0) + 1;
    });
  });

  const repeatedRecipes = Object.entries(recipeCounts)
    .filter(([recipe_id, count]) => count > 2) // Same recipe 3+ times in 4 weeks
    .map(([recipe_id]) => recipe_id);

  return {
    is_bored: overused.length > 3 || repeatedRecipes.length > 2,
    overused_ingredients: overused,
    repeated_recipes: repeatedRecipes
  };
}

// AI Prompt Adjustment
async function generateVarietyPlan(user) {
  const boredom = await detectBoredom(user.id);

  const prompt = `
    Generate a 7-day Mediterranean Keto meal plan with HIGH VARIETY.

    User is showing signs of boredom:
    - Overused ingredients (avoid these): ${boredom.overused_ingredients.join(', ')}
    - Repeated recipes (avoid): ${boredom.repeated_recipes.join(', ')}

    REQUIREMENTS:
    - Introduce NEW ingredients: ${getSeasonalIngredients(getCurrentSeason())}
    - Try different cooking methods (grilled, roasted, braised, raw)
    - Mix cuisines (Greek, Italian, Spanish, Moroccan)
    - Include 2 "exotic" Mediterranean recipes this week
    - Zero recipe repetition from past 4 weeks

    Make it exciting and fresh!
  `;

  return await openai.generate(prompt);
}

// Auto-notify user
if (boredom.is_bored) {
  await sendEmail({
    template: 'variety_boost',
    to: user.email,
    subject: '🌟 Fresh Mediterranean Flavors This Week!',
    data: {
      message: "We noticed you've been enjoying similar meals. This week we're spicing things up with new ingredients and cuisines!",
      preview_meals: ['Moroccan Lamb Tagine', 'Spanish Gambas al Ajillo', 'Greek Moussaka']
    }
  });
}
```

---

## 🔄 AUTO-ADAPTATION FLOWCHART

```
User Action
    ↓
┌───────────────────────────────────────────┐
│ What Changed?                              │
├───────────────────────────────────────────┤
│                                            │
│ 1. Blocked Ingredient (broccoli)          │
│    → Add to disliked_ingredients           │
│    → Swap current week meals (if any)     │
│    → Future weeks auto-exclude            │
│                                            │
│ 2. Changed Goal (weight loss → muscle)    │
│    → Recalculate macros                   │
│    → Regenerate current week (if not started) │
│    → Apply next Monday (if already started)│
│                                            │
│ 3. Swapped Meal (3rd time for fish)       │
│    → Detect pattern                       │
│    → Auto-reduce fish frequency           │
│    → Notify user of adjustment            │
│                                            │
│ 4. Low Engagement (2 weeks no PDF)        │
│    → Flag as at-risk churn                │
│    → Send re-engagement email             │
│    → Offer preference reset               │
│                                            │
│ 5. High Ratings for Italian Cuisine       │
│    → Increase Italian recipe %            │
│    → Silent adjustment (no notification)  │
│                                            │
└───────────────────────────────────────────┘
    ↓
Database Update (Immediate)
    ↓
Next Plan Generation (Monday 6AM)
    ↓
AI uses updated preferences
```

---

## 📊 PREFERENCE DASHBOARD (User View)

```jsx
<PreferencesDashboard>

  <Section title="My Current Profile">
    <ProfileCard>
      Goal: Muscle Gain (changed Nov 9) ✏️
      Age: 30-40
      Activity: Moderate
      Cooking Time: Max 30 minutes
    </ProfileCard>
  </Section>

  <Section title="Foods I Avoid">
    <BlockedList>
      🚫 Broccoli (blocked Nov 9)
      🚫 Anchovies (blocked Nov 5)
      🚫 Shellfish (allergy)
    </BlockedList>
    <AddButton>+ Block More</AddButton>
  </Section>

  <Section title="Meal Frequency">
    <FrequencySliders>
      Fish: 3-4 meals/week 🐟
      Vegetarian: 2 meals/week 🥗
      Red Meat: 0-1 meals/week 🥩
    </FrequencySliders>
  </Section>

  <Section title="AI Learned About You">
    <InsightCard>
      💡 You prefer quick meals (we noticed you swap complex recipes)
      💡 You love Greek cuisine (5-star ratings!)
      💡 You're very engaged (opened 8/10 emails)
    </InsightCard>
  </Section>

  <Section title="What's Changing Next Week">
    <ChangePreview>
      ✅ Your new muscle gain macros take effect Monday
      ✅ Fish reduced to 2 meals/week (based on your swaps)
      ✅ More Italian recipes (you loved them!)
    </ChangePreview>
  </Section>

</PreferencesDashboard>
```

---

## 🎯 KEY DESIGN PRINCIPLES

### 1. **Immediate Feedback**
User blocks broccoli → See confirmation instantly → Current week updated immediately

### 2. **Transparency**
Show user what AI learned: "We noticed you swap fish meals → Reduced fish to 2/week"

### 3. **Easy Undo**
Every preference has an "undo" button. Blocked broccoli by mistake? Unblock in 1 click.

### 4. **Progressive Disclosure**
Don't ask everything upfront. Learn over time from behavior.

### 5. **Smart Defaults**
If user doesn't set preferences, AI uses safe Mediterranean keto defaults.

---

## 🚨 EDGE CASES HANDLED

### "I blocked too many ingredients, now my meals are boring"

```javascript
async function validatePreferences(user_id) {
  const user = await getUser(user_id);
  const blockedCount = user.preferences.disliked_ingredients.length;

  if (blockedCount > 15) {
    return {
      warning: true,
      message: "You've blocked 15+ ingredients. This might limit meal variety. Consider unblocking some ingredients you're neutral about.",
      suggestions: user.preferences.disliked_ingredients
        .filter(i => i.reason === 'dont_like_taste') // Subjective, not allergy
        .slice(0, 5)
    };
  }
}

// Show warning in UI before saving
if (validation.warning) {
  <WarningModal>
    ⚠️ Too many blocked ingredients!
    This might make your meal plans repetitive.

    Consider unblocking:
    - Broccoli (you can swap individual meals instead)
    - Eggplant
    - Olives
  </WarningModal>
}
```

### "I changed my goal 3 times this week"

```javascript
// Rate limiting
const goalChanges = await db.audit_log.count({
  user_id,
  action: 'goal_change',
  created_at: { $gte: getWeeksAgo(1) }
});

if (goalChanges >= 3) {
  return {
    error: true,
    message: "You've changed your goal 3 times this week. Please stick with one goal for at least a week to see results.",
    can_change_after: getNextMonday()
  };
}
```

### "AI keeps giving me fish even though I blocked it"

```javascript
// Debug mode for user
GET /api/v1/preferences/debug

{
  "user_id": "usr_123",
  "current_preferences": {
    "disliked_ingredients": ["salmon", "tuna", "cod"], // User blocked these
    "meal_frequency": { "fish_per_week": "0-1" }
  },
  "last_plan_ingredients": [
    // Check if fish appeared
    { "meal": "Monday Lunch", "ingredients": ["chicken", "olives"] }, // OK
    { "meal": "Tuesday Dinner", "ingredients": ["sardines"] } // ❌ BUG!
  ],
  "ai_prompt_used": "...[avoid: salmon, tuna, cod]...", // Verify prompt was correct
  "generation_errors": []
}

// If bug found → Flag for manual review + auto-regenerate
if (foundBlockedIngredient) {
  await regenerateWeekPlan(user_id);
  await notifySupport({
    issue: 'blocked_ingredient_appeared',
    user_id,
    ingredient: 'sardines',
    plan_id: currentWeek.id
  });
}
```

---

## 📈 SUCCESS METRICS

```javascript
PREFERENCE_METRICS = {
  // Adoption
  'users_with_custom_preferences': 'Percentage',
  'avg_blocked_ingredients_per_user': 'Gauge',
  'goal_changes_per_month': 'Counter',

  // AI Learning
  'implicit_preferences_detected': 'Counter',
  'auto_adjustments_made': 'Counter', // e.g., reduced fish after 3 swaps
  'user_accepted_auto_adjustments': 'Percentage', // Did user keep the change?

  // Engagement
  'meal_swaps_per_week': 'Histogram',
  'preference_page_visits': 'Counter',
  'churn_rate_users_with_prefs_vs_default': 'Comparison',

  // Quality
  'complaints_about_ingredients': 'Counter', // "I blocked broccoli but got it"
  'plan_satisfaction_rating': 'Gauge'
};
```

---

## ✅ IMPLEMENTATION CHECKLIST

**Phase 1: Explicit Preferences (Week 1)**
- [ ] Preferences page UI
- [ ] Block ingredient API
- [ ] Change goal/cooking time API
- [ ] Meal frequency sliders
- [ ] Save preferences endpoint
- [ ] Apply preferences in AI generation

**Phase 2: Contextual Actions (Week 2)**
- [ ] Block button on ingredient list
- [ ] Swap meal modal with reasons
- [ ] Immediate swap on current week
- [ ] Email notifications for changes

**Phase 3: Implicit Learning (Week 3)**
- [ ] Track recipe ratings
- [ ] Track meal swaps with reasons
- [ ] Detect patterns (3+ swaps → auto-adjust)
- [ ] AI learning algorithm
- [ ] Transparency dashboard ("AI learned")

**Phase 4: Auto-Adaptation (Week 4)**
- [ ] Macro recalculation on goal change
- [ ] Boredom detection
- [ ] Variety boosting
- [ ] Edge case handling
- [ ] Debug mode for troubleshooting

---

**Summary:** 100% self-service. Users can change anything, anytime. AI learns silently. Zero human intervention needed. 🚀
