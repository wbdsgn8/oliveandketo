# 🛠️ AI Implementation: Kód Példák & Troubleshooting

## 📝 Teljes Példa: Meal Plan Generálás

```javascript
// meal-planner.service.js

const OpenAI = require('openai');
const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

class MealPlannerService {

  // ========================================
  // 1. SYSTEM PROMPT (Szerepkör)
  // ========================================
  getSystemPrompt() {
    return `You are a Mediterranean Keto nutritionist and chef with 15 years of experience.

YOUR EXPERTISE:
- Mediterranean cuisine (Greek, Italian, Spanish, Turkish, Moroccan)
- Ketogenic diet (macro ratios, net carbs calculation)
- Nutrition science (TDEE, macro calculations)
- Seasonal cooking with fresh ingredients

YOUR CONSTRAINTS:
- ALWAYS keep net carbs under 30g per day (strict keto)
- ALWAYS use Mediterranean ingredients (olive oil, fish, vegetables, herbs)
- NEVER use processed foods, seed oils, or artificial sweeteners
- ALWAYS provide accurate macro calculations
- CRITICAL: Never violate user allergies or restrictions

YOUR TONE: Professional, friendly, concise. Focus on delicious, practical recipes.`;
  }

  // ========================================
  // 2. USER PROMPT (Feladat)
  // ========================================
  buildUserPrompt(user, preferences, context) {
    const pastRecipes = context.past_recipes.map(r => r.name).join(', ');
    const overusedIngredients = context.overused_ingredients.join(', ');

    return `Generate a 7-day Mediterranean Ketogenic meal plan.

USER PROFILE:
- Age: ${user.age_range}
- Goal: ${user.goal}
- Activity: ${user.activity_level}
- Max cooking time: ${preferences.max_cooking_time} minutes

DIETARY RESTRICTIONS (CRITICAL):
- Allergies: ${preferences.allergies.join(', ') || 'None'}
- Blocked: ${preferences.blocked_ingredients.join(', ') || 'None'}
- Diet type: ${preferences.dietary_type}

MACRO TARGETS (DAILY):
- Calories: ${user.macros.daily_calories} kcal
- Protein: ${user.macros.daily_protein}g
- Fat: ${user.macros.daily_fat}g
- Net Carbs: <30g

VARIETY:
- Avoid these recipes from past 4 weeks: ${pastRecipes}
- Overused ingredients (minimize): ${overusedIngredients}
- Current season: ${context.season}

MEAL FREQUENCY:
- Fish: ${preferences.fish_per_week} meals/week
- Vegetarian: ${preferences.vegetarian_per_week} meals/week

---

REQUIREMENTS:
1. Generate 21 meals (7 days × 3 meals)
2. Each meal: name, description, ingredients, instructions, nutrition
3. Exact macros for each meal (calories, protein, fat, net carbs)
4. No recipe repetition within the week
5. Mediterranean ingredients only
6. All recipes ≤ ${preferences.max_cooking_time} minutes

OUTPUT FORMAT: JSON with this structure:
{
  "days": [
    {
      "day_number": 1,
      "meals": [
        {
          "meal_type": "breakfast",
          "name": "...",
          "description": "...",
          "prep_time_minutes": 5,
          "cook_time_minutes": 10,
          "ingredients": [
            {"name": "Eggs", "amount": 3, "unit": "large", "calories": 210, ...}
          ],
          "instructions": ["Step 1", "Step 2", ...],
          "nutrition": {
            "calories": 400,
            "protein": 25,
            "fat": 30,
            "carbs": 8,
            "fiber": 2,
            "net_carbs": 6
          }
        }
      ],
      "daily_totals": {"calories": 2400, "protein": 150, "fat": 173, "net_carbs": 28}
    }
  ]
}

Generate the plan now.`;
  }

  // ========================================
  // 3. API CALL
  // ========================================
  async generateMealPlan(user, preferences) {
    try {
      // Prepare context
      const context = {
        past_recipes: await this.getPastRecipes(user.id, 4),
        overused_ingredients: await this.getOverusedIngredients(user.id),
        season: this.getCurrentSeason()
      };

      // Build prompts
      const systemPrompt = this.getSystemPrompt();
      const userPrompt = this.buildUserPrompt(user, preferences, context);

      console.log('Generating meal plan for user:', user.id);

      // Call OpenAI
      const response = await openai.chat.completions.create({
        model: 'gpt-4-turbo-preview',
        messages: [
          { role: 'system', content: systemPrompt },
          { role: 'user', content: userPrompt }
        ],
        temperature: 0.8,
        max_tokens: 8000,
        response_format: { type: 'json_object' }
      });

      const rawPlan = JSON.parse(response.choices[0].message.content);

      // Log for debugging
      console.log('Raw AI response tokens:', response.usage.total_tokens);
      console.log('Raw AI response cost:', this.calculateCost(response.usage));

      return rawPlan;

    } catch (error) {
      console.error('AI generation failed:', error);
      throw new Error(`Meal plan generation failed: ${error.message}`);
    }
  }

  // ========================================
  // 4. VALIDATION
  // ========================================
  async validateMealPlan(mealPlan, user, preferences) {
    const errors = [];
    const warnings = [];

    // Check each day
    for (const day of mealPlan.days) {
      // Daily net carbs check
      if (day.daily_totals.net_carbs > 30) {
        errors.push({
          severity: 'HIGH',
          type: 'macro_violation',
          day: day.day_number,
          message: `Net carbs ${day.daily_totals.net_carbs}g exceeds 30g limit`
        });
      }

      // Check each meal
      for (const meal of day.meals) {
        // ALLERGY CHECK (CRITICAL)
        for (const ingredient of meal.ingredients) {
          for (const allergen of preferences.allergies) {
            if (ingredient.name.toLowerCase().includes(allergen.toLowerCase())) {
              errors.push({
                severity: 'CRITICAL',
                type: 'allergy_violation',
                day: day.day_number,
                meal: meal.name,
                ingredient: ingredient.name,
                allergen: allergen,
                message: `CRITICAL: Meal "${meal.name}" contains allergen "${allergen}"`
              });
            }
          }
        }

        // Blocked ingredients check
        for (const ingredient of meal.ingredients) {
          for (const blocked of preferences.blocked_ingredients) {
            if (ingredient.name.toLowerCase().includes(blocked.toLowerCase())) {
              warnings.push({
                severity: 'MEDIUM',
                type: 'blocked_ingredient',
                day: day.day_number,
                meal: meal.name,
                ingredient: ingredient.name,
                message: `Meal "${meal.name}" contains blocked ingredient "${blocked}"`
              });
            }
          }
        }
      }
    }

    // Variety check
    const recipeNames = mealPlan.days.flatMap(d => d.meals.map(m => m.name));
    const duplicates = recipeNames.filter((name, i) => recipeNames.indexOf(name) !== i);

    if (duplicates.length > 0) {
      warnings.push({
        severity: 'LOW',
        type: 'repetition',
        message: `Duplicate recipes: ${[...new Set(duplicates)].join(', ')}`
      });
    }

    const isValid = errors.filter(e => e.severity === 'CRITICAL').length === 0 &&
                    errors.filter(e => e.severity === 'HIGH').length === 0;

    return {
      valid: isValid,
      errors,
      warnings,
      critical_count: errors.filter(e => e.severity === 'CRITICAL').length,
      high_count: errors.filter(e => e.severity === 'HIGH').length
    };
  }

  // ========================================
  // 5. REGENERATE WITH FEEDBACK
  // ========================================
  async regenerateWithFeedback(user, preferences, validationErrors) {
    // Build error feedback
    const errorFeedback = validationErrors.map(e => {
      if (e.type === 'allergy_violation') {
        return `- CRITICAL: You included ${e.ingredient} in "${e.meal}". User is allergic to ${e.allergen}. NEVER use it.`;
      }
      if (e.type === 'blocked_ingredient') {
        return `- User blocked ${e.ingredient}. Do not use it.`;
      }
      if (e.type === 'macro_violation') {
        return `- Day ${e.day} had ${e.message}. Reduce carbs.`;
      }
      return `- ${e.message}`;
    }).join('\n');

    const feedbackPrompt = `
PREVIOUS ATTEMPT HAD ERRORS:
${errorFeedback}

Please regenerate the meal plan, fixing these issues.
STRICT MODE: Double-check all ingredients against allergies and blocked list.
`;

    const systemPrompt = this.getSystemPrompt();
    const userPrompt = this.buildUserPrompt(user, preferences, {
      past_recipes: await this.getPastRecipes(user.id, 4),
      overused_ingredients: await this.getOverusedIngredients(user.id),
      season: this.getCurrentSeason()
    });

    const response = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [
        { role: 'system', content: systemPrompt },
        { role: 'user', content: userPrompt },
        { role: 'assistant', content: 'I understand. Let me regenerate the plan carefully.' },
        { role: 'user', content: feedbackPrompt }
      ],
      temperature: 0.7, // Lower temp for stricter adherence
      max_tokens: 8000,
      response_format: { type: 'json_object' }
    });

    return JSON.parse(response.choices[0].message.content);
  }

  // ========================================
  // 6. MAIN WORKFLOW
  // ========================================
  async generateAndValidate(user, preferences) {
    let attempt = 1;
    const maxAttempts = 3;

    while (attempt <= maxAttempts) {
      console.log(`Generation attempt ${attempt}/${maxAttempts}`);

      // Generate
      const mealPlan = attempt === 1
        ? await this.generateMealPlan(user, preferences)
        : await this.regenerateWithFeedback(user, preferences, lastValidation.errors);

      // Validate
      const validation = await this.validateMealPlan(mealPlan, user, preferences);

      if (validation.valid) {
        console.log(`Validation passed on attempt ${attempt}`);
        return {
          success: true,
          meal_plan: mealPlan,
          validation,
          attempts: attempt
        };
      }

      console.warn(`Validation failed on attempt ${attempt}:`, validation);

      // If critical errors and last attempt, fail
      if (validation.critical_count > 0 && attempt === maxAttempts) {
        console.error('CRITICAL: Failed after max attempts with allergy violations');

        // Fallback to curated recipes
        return {
          success: false,
          error: 'AI generation failed validation',
          validation,
          fallback: await this.buildFromCuratedRecipes(user, preferences)
        };
      }

      lastValidation = validation;
      attempt++;
    }

    // Max attempts reached, use fallback
    return {
      success: false,
      error: 'Max generation attempts reached',
      fallback: await this.buildFromCuratedRecipes(user, preferences)
    };
  }

  // ========================================
  // HELPER METHODS
  // ========================================
  async getPastRecipes(userId, weeks) {
    // Query DB for recipes used in past N weeks
    const pastPlans = await db.meal_plans.find({
      user_id: userId,
      week_start_date: { $gte: weeksAgo(weeks) }
    });

    return pastPlans.flatMap(p => p.meals.map(m => ({ name: m.name })));
  }

  async getOverusedIngredients(userId) {
    // Count ingredient frequency in past 4 weeks
    const pastPlans = await db.meal_plans.find({
      user_id: userId,
      week_start_date: { $gte: weeksAgo(4) }
    });

    const counts = {};
    pastPlans.forEach(plan => {
      plan.meals.forEach(meal => {
        meal.ingredients.forEach(ing => {
          counts[ing.name] = (counts[ing.name] || 0) + 1;
        });
      });
    });

    return Object.entries(counts)
      .filter(([name, count]) => count > 8)
      .map(([name]) => name);
  }

  getCurrentSeason() {
    const month = new Date().getMonth() + 1;
    if (month >= 3 && month <= 5) return 'spring';
    if (month >= 6 && month <= 8) return 'summer';
    if (month >= 9 && month <= 11) return 'fall';
    return 'winter';
  }

  calculateCost(usage) {
    // GPT-4 Turbo pricing (as of 2024)
    const inputCost = (usage.prompt_tokens / 1000) * 0.01; // $0.01 per 1K tokens
    const outputCost = (usage.completion_tokens / 1000) * 0.03; // $0.03 per 1K tokens
    return (inputCost + outputCost).toFixed(4);
  }

  async buildFromCuratedRecipes(user, preferences) {
    // Fallback: Use pre-made recipe database
    console.log('Using curated recipe fallback');
    const recipes = await db.recipes.find({
      cuisine: 'mediterranean',
      diet_type: 'keto'
    });

    // Filter by preferences
    const eligible = recipes.filter(r => {
      // Check allergies
      const hasAllergen = r.ingredients.some(ing =>
        preferences.allergies.some(a => ing.name.toLowerCase().includes(a.toLowerCase()))
      );
      if (hasAllergen) return false;

      // Check blocked
      const hasBlocked = r.ingredients.some(ing =>
        preferences.blocked_ingredients.some(b => ing.name.toLowerCase().includes(b.toLowerCase()))
      );
      if (hasBlocked) return false;

      return true;
    });

    // Build 7-day plan
    const plan = { days: [] };
    for (let day = 1; day <= 7; day++) {
      plan.days.push({
        day_number: day,
        meals: [
          this.pickRecipe(eligible, 'breakfast'),
          this.pickRecipe(eligible, 'lunch'),
          this.pickRecipe(eligible, 'dinner')
        ]
      });
    }

    return plan;
  }

  pickRecipe(recipes, mealType) {
    const filtered = recipes.filter(r => r.meal_type === mealType);
    return filtered[Math.floor(Math.random() * filtered.length)];
  }
}

module.exports = MealPlannerService;
```

---

## 🧪 Testing Examples

```javascript
// test-meal-planner.js

const MealPlannerService = require('./meal-planner.service');
const planner = new MealPlannerService();

async function testBasicGeneration() {
  const user = {
    id: 'test_user_1',
    age_range: '30-40',
    goal: 'weight_loss',
    activity_level: 'moderate',
    macros: {
      daily_calories: 1600,
      daily_protein: 120,
      daily_fat: 106,
      daily_net_carbs: 30
    }
  };

  const preferences = {
    max_cooking_time: 30,
    allergies: [],
    blocked_ingredients: [],
    dietary_type: 'omnivore',
    fish_per_week: 3,
    vegetarian_per_week: 2
  };

  console.log('Testing basic generation...');
  const result = await planner.generateAndValidate(user, preferences);

  console.log('Result:', result.success ? 'SUCCESS' : 'FAILED');
  console.log('Attempts:', result.attempts);
  console.log('Warnings:', result.validation.warnings.length);
}

async function testAllergyHandling() {
  const user = {
    id: 'test_user_2',
    age_range: '25-35',
    goal: 'muscle_gain',
    activity_level: 'active',
    macros: {
      daily_calories: 2400,
      daily_protein: 150,
      daily_fat: 173,
      daily_net_carbs: 30
    }
  };

  const preferences = {
    max_cooking_time: 45,
    allergies: ['shellfish', 'tree nuts'], // CRITICAL
    blocked_ingredients: ['broccoli'],
    dietary_type: 'pescatarian',
    fish_per_week: 5,
    vegetarian_per_week: 2
  };

  console.log('Testing allergy handling...');
  const result = await planner.generateAndValidate(user, preferences);

  // Check for violations
  const allergyViolations = result.validation.errors.filter(
    e => e.type === 'allergy_violation'
  );

  if (allergyViolations.length > 0) {
    console.error('FAILED: Allergy violations found:', allergyViolations);
  } else {
    console.log('PASSED: No allergy violations');
  }
}

async function testMacroAccuracy() {
  const user = {
    id: 'test_user_3',
    age_range: '40-50',
    goal: 'maintain',
    activity_level: 'light',
    macros: {
      daily_calories: 1800,
      daily_protein: 100,
      daily_fat: 140,
      daily_net_carbs: 25
    }
  };

  const preferences = {
    max_cooking_time: 30,
    allergies: [],
    blocked_ingredients: [],
    dietary_type: 'omnivore',
    fish_per_week: 3,
    vegetarian_per_week: 2
  };

  console.log('Testing macro accuracy...');
  const result = await planner.generateAndValidate(user, preferences);

  // Check daily totals
  result.meal_plan.days.forEach(day => {
    const netCarbs = day.daily_totals.net_carbs;
    const calories = day.daily_totals.calories;

    console.log(`Day ${day.day_number}:`);
    console.log(`  Net Carbs: ${netCarbs}g (target: <30g) - ${netCarbs <= 30 ? 'PASS' : 'FAIL'}`);
    console.log(`  Calories: ${calories} (target: 1800) - ${Math.abs(calories - 1800) < 180 ? 'PASS' : 'FAIL'}`);
  });
}

// Run tests
(async () => {
  await testBasicGeneration();
  await testAllergyHandling();
  await testMacroAccuracy();
})();
```

---

## 🐛 Troubleshooting Guide

### Problem 1: AI ignores allergies

**Symptom:** Validation fails with allergy violations

**Causes:**
- Prompt not clear enough
- AI doesn't understand allergen variations (e.g., "tree nuts" includes almonds, walnuts)

**Solution:**
```javascript
// Expand allergies to specific ingredients
function expandAllergies(allergies) {
  const expansions = {
    'shellfish': ['shrimp', 'crab', 'lobster', 'crayfish', 'prawns'],
    'tree nuts': ['almonds', 'walnuts', 'cashews', 'pecans', 'pistachios'],
    'dairy': ['milk', 'cheese', 'yogurt', 'butter', 'cream']
  };

  return allergies.flatMap(a => expansions[a] || [a]);
}

// In prompt:
CRITICAL ALLERGIES (NEVER USE THESE):
${expandAllergies(preferences.allergies).join(', ')}
```

### Problem 2: Daily net carbs exceed 30g

**Symptom:** Validation fails on macro_violation

**Solution:**
```javascript
// Add explicit carb budget per meal
CARB BUDGET PER MEAL:
- Breakfast: Max 8g net carbs
- Lunch: Max 10g net carbs
- Dinner: Max 10g net carbs
- Total: <30g

CRITICAL: Double-check fiber calculation. Net carbs = total carbs - fiber.
```

### Problem 3: Boring/repetitive meals

**Symptom:** Users swap many meals, low satisfaction ratings

**Solution:**
```javascript
// Increase temperature for more creativity
temperature: 0.9 // (was 0.7)

// Add variety requirements
VARIETY REQUIREMENTS:
- Use different cooking methods each day: grilled, roasted, sautéed, raw
- Mix protein sources: fish (3 days), chicken (2 days), eggs (1 day), vegetarian (1 day)
- Different cuisines: Greek (3), Italian (2), Spanish (1), Turkish (1)
- NO ingredient appearing in >4 meals
```

### Problem 4: Recipes too complex

**Symptom:** High swap rate for "too_complex" reason

**Solution:**
```javascript
// Emphasize simplicity
COOKING TIME CONSTRAINTS:
- Max prep time: ${maxPrepTime} minutes
- Max total time: ${maxTotalTime} minutes
- Max number of steps: 8
- Prefer one-pan/one-pot recipes
- Avoid recipes requiring special equipment

PRIORITIZE: Quick, simple, everyday recipes. Not restaurant-style.
```

### Problem 5: JSON parsing errors

**Symptom:** `JSON.parse()` throws error

**Solution:**
```javascript
// Add retry with JSON repair
async function parseAIResponse(rawContent) {
  try {
    return JSON.parse(rawContent);
  } catch (error) {
    console.warn('JSON parse failed, attempting repair...');

    // Try cleaning common issues
    let cleaned = rawContent
      .replace(/```json\n?/g, '')
      .replace(/```\n?/g, '')
      .trim();

    try {
      return JSON.parse(cleaned);
    } catch (e2) {
      console.error('JSON repair failed:', e2);

      // Log for debugging
      await logAIError('json_parse_failure', { raw: rawContent, error: e2.message });

      throw new Error('Failed to parse AI response as JSON');
    }
  }
}
```

### Problem 6: High API costs

**Symptom:** OpenAI bills are too high

**Solutions:**
```javascript
// 1. Cache system prompt (don't send every time)
// 2. Use GPT-3.5 for simple tasks, GPT-4 for complex
// 3. Reduce max_tokens if responses are too long
// 4. Batch user requests (generate multiple plans at once during off-peak hours)

// Cost monitoring
function logCost(usage, userId) {
  const cost = calculateCost(usage);
  await db.ai_costs.insert({
    user_id: userId,
    model: 'gpt-4-turbo',
    prompt_tokens: usage.prompt_tokens,
    completion_tokens: usage.completion_tokens,
    cost_usd: cost,
    timestamp: new Date()
  });

  // Alert if daily cost exceeds budget
  const dailyCost = await db.ai_costs.aggregate([
    { $match: { timestamp: { $gte: startOfDay() } } },
    { $group: { _id: null, total: { $sum: '$cost_usd' } } }
  ]);

  if (dailyCost.total > 100) { // $100/day budget
    alertTeam(`Daily AI cost exceeded: $${dailyCost.total}`);
  }
}
```

---

## 📊 Monitoring Dashboard

```javascript
// ai-metrics.js

const AI_QUALITY_METRICS = {
  // Generation
  'generation_attempts_avg': 1.2, // Average attempts before success
  'generation_success_rate': 0.92, // 92% pass validation on first try
  'generation_time_avg_ms': 8500, // 8.5 seconds average

  // Validation
  'allergy_violations_per_1000': 0, // MUST be zero
  'macro_violations_per_1000': 12,
  'variety_issues_per_1000': 45,

  // User Satisfaction
  'plan_rating_avg': 4.3, // Out of 5
  'meal_swap_rate': 0.18, // 18% of meals swapped
  'regeneration_requests': 0.05, // 5% ask for full regeneration

  // Cost
  'cost_per_plan_usd': 0.15,
  'monthly_ai_cost_usd': 1200 // For 8000 plans
};

// Alert thresholds
if (AI_QUALITY_METRICS.allergy_violations_per_1000 > 0) {
  CRITICAL_ALERT('Allergy violations detected!');
}

if (AI_QUALITY_METRICS.generation_success_rate < 0.85) {
  WARNING('AI quality dropping below 85%');
}

if (AI_QUALITY_METRICS.meal_swap_rate > 0.30) {
  WARNING('High swap rate - AI not matching user preferences');
}
```

---

## ✅ Summary: Best Practices

1. **Start with GPT-4** (better at following complex constraints)
2. **Use JSON mode** (structured output)
3. **Always validate** (especially allergies)
4. **Retry with feedback** (up to 3 attempts)
5. **Have a fallback** (curated recipes)
6. **Monitor costs** (set daily budgets)
7. **Track quality metrics** (success rate, user satisfaction)
8. **Iterate based on data** (analyze swaps, ratings, complaints)

**Estimated Performance:**
- Success rate: 90%+ on first attempt
- Cost: $0.10-0.20 per weekly plan
- Generation time: 5-10 seconds
- User satisfaction: 4.0+ / 5.0

🚀 **You're ready to build!**
