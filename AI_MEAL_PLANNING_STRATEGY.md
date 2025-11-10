# 🤖 AI Meal Planning: Prompt Engineering & Training Strategy

## 📋 Cél

**Megtanítani az AI-t arra, hogy:**
1. Mediterrán Keto recepteket generáljon
2. Követje a makró célokat (keto: <30g net carbs, 70% fat, 25% protein)
3. Mediterrán alapanyagokat használjon (olive oil, fish, vegetables, herbs)
4. Változatos, finom, könnyen főzhető ételeket készítsen
5. Személyre szabott legyen (user profile alapján)

---

## 🎯 Stratégia: Prompt Engineering (NEM Fine-tuning)

### Miért NEM kell fine-tuning?

❌ **Fine-tuning problémák:**
- Drága ($$$)
- Lassú (hetekig tart)
- Nehéz frissíteni (új alapanyag, új season → újra train)
- Túl komplex karbantartás

✅ **Prompt Engineering előnyök:**
- Azonnal működik
- Olcsó (csak API hívások)
- Könnyen frissíthető (prompt változtatás)
- Flexibilis (user preferences on-the-fly)
- GPT-4 / Claude már ismeri a mediterrán konyháit

---

## 🏗️ Multi-Layer Prompt Architecture

### Layer 1: System Prompt (Role & Constraints)

```javascript
const SYSTEM_PROMPT = `
You are a Mediterranean Keto nutritionist and chef with 15 years of experience.

YOUR EXPERTISE:
- Mediterranean cuisine (Greek, Italian, Spanish, Turkish, Moroccan)
- Ketogenic diet principles (macro ratios, net carbs calculation)
- Nutrition science (Mifflin-St Jeor equation, TDEE calculation)
- Seasonal cooking (using fresh, local ingredients)
- Meal planning for various goals (weight loss, maintenance, muscle gain)

YOUR CONSTRAINTS:
- ALWAYS keep net carbs under 30g per day (strictly keto)
- ALWAYS use Mediterranean ingredients (olive oil, fish, vegetables, herbs, cheese, nuts)
- NEVER use processed foods, seed oils, or artificial sweeteners
- ALWAYS provide accurate macro calculations (calories, protein, fat, carbs, fiber)
- ALWAYS consider user allergies and restrictions (CRITICAL for safety)

YOUR TONE:
- Professional but friendly
- Concise and clear
- Focus on delicious, practical recipes
- Encourage healthy eating habits

IMPORTANT: If a request violates keto or Mediterranean principles, explain why and offer an alternative.
`;
```

---

### Layer 2: Task-Specific Prompt (Meal Plan Generation)

```javascript
async function generateWeeklyMealPlan(userProfile, userPreferences) {

  const prompt = `
Generate a 7-day Mediterranean Ketogenic meal plan for the following user:

USER PROFILE:
- Age: ${userProfile.age_range}
- Gender: ${userProfile.gender || 'Not specified'}
- Goal: ${userProfile.goal} (weight loss / maintain / muscle gain)
- Activity Level: ${userProfile.activity_level}
- Cooking Time: Maximum ${userProfile.max_cooking_time} minutes per meal

DIETARY RESTRICTIONS (CRITICAL):
- Allergies: ${userPreferences.allergies.join(', ') || 'None'}
- Intolerances: ${userPreferences.intolerances.join(', ') || 'None'}
- Dietary Type: ${userPreferences.dietary_type} (omnivore / pescatarian / vegetarian)
- Blocked Ingredients: ${userPreferences.blocked_ingredients.join(', ') || 'None'}

MEAL FREQUENCY PREFERENCES:
- Fish meals per week: ${userPreferences.fish_per_week}
- Vegetarian meals per week: ${userPreferences.vegetarian_per_week}
- Red meat per week: ${userPreferences.red_meat_per_week}

MACRO TARGETS (DAILY):
- Calories: ${userProfile.macros.daily_calories} kcal
- Protein: ${userProfile.macros.daily_protein}g
- Fat: ${userProfile.macros.daily_fat}g
- Net Carbs: <30g (strict keto limit)

VARIETY REQUIREMENTS:
- Avoid recipes from the past 4 weeks: ${getPastRecipeNames(userProfile.user_id, 4)}
- Overused ingredients (use sparingly): ${getOverusedIngredients(userProfile.user_id)}
- Current season: ${getCurrentSeason()} (prefer seasonal produce)

CUISINE PREFERENCES:
- Loved cuisines (emphasize): ${userPreferences.loved_cuisines.join(', ')}
- Disliked cuisines (minimize): ${userPreferences.disliked_cuisines.join(', ')}

---

REQUIREMENTS:

1. STRUCTURE:
   - Generate 21 meals (7 days × 3 meals: breakfast, lunch, dinner)
   - Each day should have varied macro distribution
   - Breakfast: Lighter (300-400 cal)
   - Lunch: Moderate (400-600 cal)
   - Dinner: Heartier (500-700 cal)

2. MACRO ACCURACY:
   - Each meal MUST have exact macros (calories, protein, fat, net carbs)
   - Daily totals MUST be within ±10% of target
   - Net carbs per meal MUST be <10g (to stay under 30g daily)

3. INGREDIENTS:
   - Primary fats: Extra virgin olive oil, avocados, nuts, cheese
   - Proteins: Fish (salmon, sardines, tuna), chicken, eggs, Greek yogurt
   - Vegetables: Leafy greens, tomatoes, zucchini, eggplant, peppers, olives
   - Herbs: Basil, oregano, thyme, rosemary, parsley
   - NO: Bread, pasta, rice, potatoes, sugar, processed foods

4. VARIETY:
   - NO meal repetition within the week
   - NO ingredient appearing in >5 meals
   - Mix cooking methods: grilled, roasted, sautéed, raw (salads)
   - Mix cuisines: Greek (3-4 meals), Italian (3-4), Spanish (2), Turkish (1-2)

5. PRACTICALITY:
   - All recipes ≤ ${userProfile.max_cooking_time} minutes
   - Common Mediterranean ingredients (no exotic items)
   - Clear step-by-step instructions (5-8 steps max)
   - Serving size: 1 person (scalable)

6. RECIPE FORMAT:
   For each meal, provide:
   - Name (appetizing, descriptive)
   - Description (1 sentence, make it sound delicious)
   - Prep time (minutes)
   - Cook time (minutes)
   - Serving size
   - Ingredients list (exact amounts: grams, ml, pieces)
   - Step-by-step instructions (numbered, concise)
   - Nutrition facts (calories, protein, fat, carbs, fiber, net carbs)
   - Tags: [cuisine_type, meal_type, cooking_method, main_ingredient]

---

OUTPUT FORMAT:

Return a JSON object with this structure:

{
  "week_start_date": "2025-11-11",
  "user_profile_summary": {
    "goal": "muscle_gain",
    "daily_calories": 2400,
    "daily_protein": 150,
    "daily_fat": 173,
    "daily_net_carbs": 30
  },
  "days": [
    {
      "day_number": 1,
      "date": "2025-11-11",
      "meals": [
        {
          "meal_type": "breakfast",
          "name": "Greek Yogurt Bowl with Walnuts",
          "description": "Creamy Greek yogurt topped with crunchy walnuts and a drizzle of olive oil",
          "prep_time_minutes": 5,
          "cook_time_minutes": 0,
          "total_time_minutes": 5,
          "servings": 1,
          "ingredients": [
            {
              "name": "Greek yogurt (full-fat)",
              "amount": 200,
              "unit": "g",
              "calories": 200,
              "protein": 10,
              "fat": 15,
              "carbs": 8,
              "fiber": 0
            },
            {
              "name": "Walnuts",
              "amount": 30,
              "unit": "g",
              "calories": 196,
              "protein": 5,
              "fat": 20,
              "carbs": 4,
              "fiber": 2
            },
            {
              "name": "Extra virgin olive oil",
              "amount": 10,
              "unit": "ml",
              "calories": 88,
              "protein": 0,
              "fat": 10,
              "carbs": 0,
              "fiber": 0
            },
            {
              "name": "Cinnamon",
              "amount": 1,
              "unit": "pinch",
              "calories": 0,
              "protein": 0,
              "fat": 0,
              "carbs": 0,
              "fiber": 0
            }
          ],
          "instructions": [
            "Add Greek yogurt to a bowl",
            "Top with crushed walnuts",
            "Drizzle olive oil over the top",
            "Sprinkle with a pinch of cinnamon",
            "Serve immediately"
          ],
          "nutrition": {
            "calories": 484,
            "protein": 15,
            "fat": 45,
            "carbs": 12,
            "fiber": 2,
            "net_carbs": 10,
            "fat_percentage": 84,
            "protein_percentage": 12,
            "carb_percentage": 4
          },
          "tags": ["greek", "breakfast", "no-cook", "yogurt", "quick"],
          "difficulty": "easy",
          "cuisine": "Greek"
        },
        // ... lunch and dinner for day 1
      ],
      "daily_totals": {
        "calories": 2380,
        "protein": 148,
        "fat": 171,
        "net_carbs": 28
      }
    },
    // ... days 2-7
  ],
  "weekly_totals": {
    "avg_daily_calories": 2395,
    "avg_daily_protein": 149,
    "avg_daily_fat": 172,
    "avg_daily_net_carbs": 27,
    "variance_from_target": {
      "calories": "+0.2%",
      "protein": "-0.7%",
      "fat": "-0.6%",
      "net_carbs": "-10%"
    }
  },
  "shopping_list": [
    // Generated separately (see below)
  ]
}

---

CRITICAL REMINDERS:
1. ALLERGIES: If user is allergic to shellfish, NEVER include shrimp, crab, lobster, etc.
2. BLOCKED INGREDIENTS: If user blocked "broccoli", do NOT use it in ANY meal.
3. MACRO VALIDATION: Double-check that daily net carbs stay under 30g.
4. MEDITERRANEAN: Every meal must use Mediterranean ingredients and techniques.
5. VARIETY: Check that no recipe appears in user's past 4 weeks.

Generate the meal plan now.
`;

  const response = await openai.chat.completions.create({
    model: "gpt-4",
    messages: [
      { role: "system", content: SYSTEM_PROMPT },
      { role: "user", content: prompt }
    ],
    temperature: 0.8, // Creative but not too random
    max_tokens: 8000,
    response_format: { type: "json_object" } // Force JSON output
  });

  return JSON.parse(response.choices[0].message.content);
}
```

---

## 🔍 Post-Generation Validation

**KRITIKUS: Mindig validáld a generált tervet!**

```javascript
async function validateMealPlan(mealPlan, userProfile, userPreferences) {
  const errors = [];
  const warnings = [];

  // 1. ALLERGY CHECK (CRITICAL - Can't fail)
  for (const day of mealPlan.days) {
    for (const meal of day.meals) {
      for (const ingredient of meal.ingredients) {
        // Check allergies
        if (userPreferences.allergies.some(allergy =>
          ingredient.name.toLowerCase().includes(allergy.toLowerCase())
        )) {
          errors.push({
            severity: "CRITICAL",
            type: "allergy_violation",
            day: day.day_number,
            meal: meal.name,
            ingredient: ingredient.name,
            allergen: userPreferences.allergies.find(a =>
              ingredient.name.toLowerCase().includes(a.toLowerCase())
            )
          });
        }

        // Check blocked ingredients
        if (userPreferences.blocked_ingredients.some(blocked =>
          ingredient.name.toLowerCase().includes(blocked.toLowerCase())
        )) {
          warnings.push({
            severity: "HIGH",
            type: "blocked_ingredient",
            day: day.day_number,
            meal: meal.name,
            ingredient: ingredient.name
          });
        }
      }
    }
  }

  // 2. MACRO VALIDATION
  for (const day of mealPlan.days) {
    const dailyNetCarbs = day.daily_totals.net_carbs;

    if (dailyNetCarbs > 30) {
      errors.push({
        severity: "HIGH",
        type: "macro_violation",
        day: day.day_number,
        issue: `Daily net carbs (${dailyNetCarbs}g) exceeds keto limit (30g)`
      });
    }

    // Check macro distribution
    const calorieTarget = userProfile.macros.daily_calories;
    const actualCalories = day.daily_totals.calories;
    const variance = Math.abs(actualCalories - calorieTarget) / calorieTarget;

    if (variance > 0.15) { // More than 15% off target
      warnings.push({
        severity: "MEDIUM",
        type: "macro_variance",
        day: day.day_number,
        issue: `Calories off by ${(variance * 100).toFixed(0)}% (target: ${calorieTarget}, actual: ${actualCalories})`
      });
    }
  }

  // 3. VARIETY CHECK
  const recipeNames = mealPlan.days.flatMap(d =>
    d.meals.map(m => m.name.toLowerCase())
  );
  const duplicates = recipeNames.filter((name, index) =>
    recipeNames.indexOf(name) !== index
  );

  if (duplicates.length > 0) {
    warnings.push({
      severity: "LOW",
      type: "repetition",
      issue: `Duplicate recipes: ${[...new Set(duplicates)].join(', ')}`
    });
  }

  // 4. INGREDIENT FREQUENCY
  const ingredientCount = {};
  mealPlan.days.forEach(day => {
    day.meals.forEach(meal => {
      meal.ingredients.forEach(ing => {
        const name = ing.name.toLowerCase();
        ingredientCount[name] = (ingredientCount[name] || 0) + 1;
      });
    });
  });

  const overused = Object.entries(ingredientCount)
    .filter(([name, count]) => count > 8) // More than 8 times in 21 meals
    .map(([name]) => name);

  if (overused.length > 0) {
    warnings.push({
      severity: "LOW",
      type: "ingredient_overuse",
      issue: `Overused ingredients: ${overused.join(', ')}`
    });
  }

  // RESULT
  const isValid = errors.length === 0;

  return {
    valid: isValid,
    errors,
    warnings,
    summary: {
      critical_errors: errors.filter(e => e.severity === "CRITICAL").length,
      high_errors: errors.filter(e => e.severity === "HIGH").length,
      total_warnings: warnings.length
    }
  };
}

// Usage
const mealPlan = await generateWeeklyMealPlan(user, preferences);
const validation = await validateMealPlan(mealPlan, user, preferences);

if (!validation.valid) {
  if (validation.summary.critical_errors > 0) {
    // CRITICAL: Allergies found! MUST regenerate
    console.error("CRITICAL ERRORS:", validation.errors);

    // Regenerate with stricter prompt
    const fixedPlan = await regenerateMealPlan(user, preferences, {
      strict_mode: true,
      previous_errors: validation.errors
    });
  } else {
    // Minor issues, can proceed with warnings
    console.warn("Warnings:", validation.warnings);
  }
}
```

---

## 📚 Knowledge Base: Recipe Examples (Few-Shot Learning)

**Adj az AI-nek példa recepteket a prompt-ban:**

```javascript
const RECIPE_EXAMPLES = `
Here are 3 example recipes to follow the format:

EXAMPLE 1 - Breakfast:
{
  "name": "Greek Feta Omelette with Olives",
  "description": "Fluffy eggs filled with tangy feta and Mediterranean olives",
  "prep_time_minutes": 5,
  "cook_time_minutes": 8,
  "ingredients": [
    {"name": "Eggs", "amount": 3, "unit": "large"},
    {"name": "Feta cheese", "amount": 40, "unit": "g"},
    {"name": "Kalamata olives", "amount": 30, "unit": "g"},
    {"name": "Extra virgin olive oil", "amount": 15, "unit": "ml"},
    {"name": "Fresh oregano", "amount": 5, "unit": "g"}
  ],
  "instructions": [
    "Beat 3 eggs in a bowl with a pinch of salt",
    "Heat olive oil in a non-stick pan over medium heat",
    "Pour eggs into the pan, let set for 2 minutes",
    "Add crumbled feta and sliced olives to one half",
    "Fold omelette in half, cook 2 more minutes",
    "Garnish with fresh oregano and serve"
  ],
  "nutrition": {
    "calories": 420,
    "protein": 24,
    "fat": 35,
    "net_carbs": 3
  }
}

EXAMPLE 2 - Lunch:
{
  "name": "Mediterranean Tuna Salad with Avocado",
  "description": "Fresh tuna mixed with creamy avocado, tomatoes, and olive oil",
  "prep_time_minutes": 10,
  "cook_time_minutes": 0,
  "ingredients": [
    {"name": "Canned tuna in olive oil", "amount": 150, "unit": "g"},
    {"name": "Avocado", "amount": 100, "unit": "g"},
    {"name": "Cherry tomatoes", "amount": 100, "unit": "g"},
    {"name": "Cucumber", "amount": 80, "unit": "g"},
    {"name": "Red onion", "amount": 30, "unit": "g"},
    {"name": "Extra virgin olive oil", "amount": 20, "unit": "ml"},
    {"name": "Lemon juice", "amount": 15, "unit": "ml"},
    {"name": "Fresh parsley", "amount": 10, "unit": "g"}
  ],
  "instructions": [
    "Drain tuna and flake into a large bowl",
    "Dice avocado, tomatoes, cucumber, and red onion",
    "Add all vegetables to the bowl with tuna",
    "Drizzle with olive oil and lemon juice",
    "Toss gently to combine",
    "Garnish with fresh parsley and serve"
  ],
  "nutrition": {
    "calories": 480,
    "protein": 32,
    "fat": 38,
    "net_carbs": 6
  }
}

EXAMPLE 3 - Dinner:
{
  "name": "Grilled Salmon with Lemon & Herbs",
  "description": "Perfectly grilled salmon with fresh lemon and Mediterranean herbs",
  "prep_time_minutes": 10,
  "cook_time_minutes": 15,
  "ingredients": [
    {"name": "Salmon fillet", "amount": 200, "unit": "g"},
    {"name": "Extra virgin olive oil", "amount": 20, "unit": "ml"},
    {"name": "Lemon", "amount": 1, "unit": "whole"},
    {"name": "Fresh dill", "amount": 10, "unit": "g"},
    {"name": "Garlic cloves", "amount": 2, "unit": "cloves"},
    {"name": "Spinach", "amount": 150, "unit": "g"},
    {"name": "Cherry tomatoes", "amount": 100, "unit": "g"}
  ],
  "instructions": [
    "Preheat grill or grill pan to medium-high heat",
    "Brush salmon with olive oil, season with salt and pepper",
    "Place salmon skin-side down on grill, cook 6-7 minutes",
    "Flip and cook another 6-7 minutes until flaky",
    "Meanwhile, sauté spinach and tomatoes in olive oil with garlic",
    "Plate salmon with vegetables, garnish with dill and lemon wedges"
  ],
  "nutrition": {
    "calories": 520,
    "protein": 42,
    "fat": 38,
    "net_carbs": 5
  }
}

Now generate meals following this exact format and quality.
`;
```

---

## 🧪 Iterative Refinement Strategy

### Approach 1: Single-Shot (Fast but risky)
```javascript
// Generate entire week in 1 API call
const mealPlan = await generateWeeklyMealPlan(user, prefs);
const validation = await validateMealPlan(mealPlan, user, prefs);

if (!validation.valid) {
  // Regenerate with error feedback
  const fixedPlan = await regenerateWithFeedback(user, prefs, validation.errors);
}
```

### Approach 2: Multi-Shot (Slower but safer)
```javascript
// Generate day-by-day, track running totals
const mealPlan = { days: [] };

for (let day = 1; day <= 7; day++) {
  const dayPlan = await generateDayMeals(user, prefs, {
    day_number: day,
    previous_days: mealPlan.days, // Avoid repetition
    running_totals: calculateRunningTotals(mealPlan.days)
  });

  // Validate each day immediately
  const validation = validateDay(dayPlan, user, prefs);

  if (validation.valid) {
    mealPlan.days.push(dayPlan);
  } else {
    // Retry this day with stricter constraints
    day--; // Retry same day
  }
}
```

**Recommendation:** Use Single-Shot for speed, fallback to Multi-Shot if validation fails repeatedly.

---

## 💾 Curated Recipe Database (Fallback)

**Ha az AI generálás fail-el, használj pre-made recepteket:**

```javascript
// recipes_db.json
{
  "recipes": [
    {
      "id": "rec_001",
      "name": "Greek Yogurt Bowl",
      "meal_type": "breakfast",
      "cuisine": "greek",
      "prep_time": 5,
      "cook_time": 0,
      "difficulty": "easy",
      "nutrition": {
        "calories": 320,
        "protein": 22,
        "fat": 24,
        "net_carbs": 8
      },
      "ingredients": [...],
      "instructions": [...],
      "tags": ["quick", "no-cook", "vegetarian"]
    },
    // ... 200+ curated recipes
  ]
}

// Fallback logic
async function generateMealPlanWithFallback(user, prefs) {
  try {
    // Try AI generation first
    const aiPlan = await generateWeeklyMealPlan(user, prefs);
    const validation = await validateMealPlan(aiPlan, user, prefs);

    if (validation.valid) {
      return aiPlan;
    }
  } catch (error) {
    console.error("AI generation failed:", error);
  }

  // Fallback to curated recipes
  console.log("Using curated recipe database");
  return await buildPlanFromCuratedRecipes(user, prefs);
}

async function buildPlanFromCuratedRecipes(user, prefs) {
  const recipes = await loadRecipeDatabase();

  // Filter by preferences
  let eligible = recipes.filter(r => {
    // Check allergies
    const hasAllergen = r.ingredients.some(ing =>
      prefs.allergies.some(allergy => ing.name.includes(allergy))
    );
    if (hasAllergen) return false;

    // Check blocked ingredients
    const hasBlocked = r.ingredients.some(ing =>
      prefs.blocked_ingredients.some(blocked => ing.name.includes(blocked))
    );
    if (hasBlocked) return false;

    // Check dietary type
    if (prefs.dietary_type === 'vegetarian' && r.tags.includes('meat')) {
      return false;
    }

    return true;
  });

  // Build plan (21 meals)
  const plan = { days: [] };

  for (let day = 1; day <= 7; day++) {
    const dayMeals = {
      day_number: day,
      meals: []
    };

    // Pick breakfast (avoid repetition)
    const breakfast = pickRandomRecipe(eligible, 'breakfast', plan);
    dayMeals.meals.push(breakfast);

    // Pick lunch
    const lunch = pickRandomRecipe(eligible, 'lunch', plan);
    dayMeals.meals.push(lunch);

    // Pick dinner
    const dinner = pickRandomRecipe(eligible, 'dinner', plan);
    dayMeals.meals.push(dinner);

    plan.days.push(dayMeals);
  }

  return plan;
}
```

---

## 📊 Quality Metrics & Monitoring

```javascript
// Track AI generation quality
const AI_METRICS = {
  'generation_success_rate': 'Percentage', // How many pass validation?
  'avg_generation_time_ms': 'Gauge',
  'validation_failure_reasons': 'Counter', // Why do they fail?
  'user_satisfaction_rating': 'Gauge', // User rates the plan (1-5)
  'meal_swap_rate': 'Percentage', // How many meals get swapped?
  'plan_regeneration_requests': 'Counter' // User clicks "regenerate"
};

// Alert if quality drops
if (metrics.generation_success_rate < 0.85) {
  alertTeam("AI generation quality dropped below 85%");
}

if (metrics.meal_swap_rate > 0.30) {
  alertTeam("Users swapping 30%+ of meals - AI not matching preferences");
}
```

---

## 🎯 Success Criteria

**Egy jó AI-generált meal plan:**

✅ **Macro Accuracy:** Daily net carbs <30g, within ±10% of calorie/protein/fat targets
✅ **Mediterranean:** 100% of meals use Mediterranean ingredients/techniques
✅ **Variety:** Zero recipe repetition within week, max 5 appearances per ingredient
✅ **Safety:** Zero allergen violations, zero blocked ingredients
✅ **Taste:** User ratings >4.0/5.0 average
✅ **Practicality:** All recipes ≤ user's max cooking time
✅ **Freshness:** No recipes from user's past 4 weeks

---

## 🚀 Implementation Roadmap

**Week 1:** System Prompt + Basic Meal Generation
- Write system prompt
- Test with 10 sample user profiles
- Measure macro accuracy

**Week 2:** Validation Layer
- Build validation function
- Test allergy detection (CRITICAL)
- Add regeneration logic

**Week 3:** Few-Shot Learning
- Add recipe examples to prompt
- Test variety improvements
- Monitor ingredient repetition

**Week 4:** Curated Fallback
- Build recipe database (50 recipes)
- Test fallback logic
- Compare AI vs curated quality

**Week 5+:** Iterate Based on User Feedback
- Collect user ratings
- Analyze swap patterns
- Refine prompts based on data

---

## 💡 Pro Tips

1. **Temperature:** Use 0.7-0.9 for creativity (variety) but not too random
2. **Max Tokens:** 8000+ needed for full week plan
3. **Model:** GPT-4 > GPT-3.5 (much better at following constraints)
4. **JSON Mode:** Always use `response_format: json_object` for structured output
5. **Retries:** If generation fails, retry with stricter prompt (add examples of errors)
6. **Caching:** Cache common prompt parts (system prompt, examples) to save tokens
7. **Cost:** ~$0.10-0.20 per weekly plan (GPT-4), scale to 10k users = $1k-2k/week

---

**Bottom Line:** Prompt Engineering >> Fine-tuning. GPT-4/Claude already knows Mediterranean & Keto. Just give it clear instructions, examples, and strict validation. 🚀
