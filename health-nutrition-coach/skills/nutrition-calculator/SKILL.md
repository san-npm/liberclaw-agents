# Skill: Nutrition Calculator

Provides TDEE/BMR formulas, macro calculations, meal plan structure, and grocery list generation for the Health & Nutrition Coach agent.

---

## BMR Formulas

### Mifflin-St Jeor (Preferred — most accurate for most adults)

**Male:**
```
BMR = (10 × weight_kg) + (6.25 × height_cm) − (5 × age) + 5
```

**Female:**
```
BMR = (10 × weight_kg) + (6.25 × height_cm) − (5 × age) − 161
```

### Harris-Benedict (Revised — older but widely cited)

**Male:**
```
BMR = 88.362 + (13.397 × weight_kg) + (4.799 × height_cm) − (5.677 × age)
```

**Female:**
```
BMR = 447.593 + (9.247 × weight_kg) + (3.098 × height_cm) − (4.330 × age)
```

> 💡 **Use Mifflin-St Jeor by default.** Use Harris-Benedict for cross-check if user reports BMR seems off.

---

## TDEE — Activity Multipliers

Apply to BMR based on user's actual weekly activity:

| Activity Level | Multiplier | Description |
|---|---|---|
| Sedentary | × 1.2 | Desk job, no exercise |
| Lightly active | × 1.375 | 1–3 days/week light exercise |
| Moderately active | × 1.55 | 3–5 days/week moderate exercise |
| Very active | × 1.725 | 6–7 days/week hard training |
| Extremely active | × 1.9 | Physical job + daily hard training |

```
TDEE = BMR × activity_multiplier
```

---

## Calorie Targets by Goal

| Goal | Adjustment | Notes |
|---|---|---|
| Weight loss (slow) | TDEE − 250 kcal | ~0.25 kg/week loss |
| Weight loss (moderate) | TDEE − 500 kcal | ~0.5 kg/week loss (sustainable) |
| Weight loss (aggressive) | TDEE − 750 kcal | Max for most; monitor energy levels |
| Maintenance | TDEE | No change in body weight |
| Muscle gain (lean bulk) | TDEE + 200–300 kcal | Minimize fat gain |
| Muscle gain (bulk) | TDEE + 400–500 kcal | Faster gain, more fat accumulation |

> ⚠️ **Floor:** Never recommend below 1200 kcal/day for women or 1500 kcal/day for men without medical supervision.

---

## Macro Calculations

### Protein (highest priority — set first)
| Goal | Target |
|---|---|
| Weight loss | 1.8–2.2 g/kg body weight (preserve muscle) |
| Maintenance | 1.4–1.8 g/kg |
| Muscle gain | 1.8–2.4 g/kg |
| High training volume | Up to 2.5 g/kg |

> 1 g protein = 4 kcal

### Fat (set second)
| Goal | Target |
|---|---|
| General | 20–35% of total calories |
| Low-carb / keto | 60–70% of total calories |
| Minimum for hormonal health | ≥0.8 g/kg body weight |

> 1 g fat = 9 kcal

### Carbohydrates (fill remaining calories)
```
Carb kcal = Total kcal − (Protein kcal + Fat kcal)
Carbs (g) = Carb kcal ÷ 4
```

> 1 g carbohydrate = 4 kcal

### Example Calculation
User: 80 kg male, 30 years old, 178 cm, moderately active, goal = muscle gain

```
BMR = (10 × 80) + (6.25 × 178) − (5 × 30) + 5 = 1,887.5 kcal
TDEE = 1,887.5 × 1.55 = 2,926 kcal
Target = TDEE + 300 = 3,226 kcal

Protein: 2.0 g × 80 kg = 160 g → 640 kcal
Fat: 25% of 3,226 = 806 kcal → 90 g
Carbs: 3,226 − 640 − 806 = 1,780 kcal → 445 g

Summary: 3,226 kcal | 160g P | 445g C | 90g F
```

---

## Macro Split Templates

| Diet Style | Protein | Carbs | Fat |
|---|---|---|---|
| Standard balanced | 25–30% | 40–50% | 25–35% |
| Low-carb | 30–40% | 20–30% | 35–45% |
| Ketogenic | 25–35% | 5–10% | 60–70% |
| High-carb (endurance) | 15–20% | 55–65% | 20–25% |
| High-protein (recomp) | 35–40% | 30–40% | 25–30% |

---

## Meal Plan Structure Templates

### 3 Meals / Day
```
Breakfast:  25% of daily calories
Lunch:      35% of daily calories
Dinner:     35% of daily calories
(Adjust 5% for a small dessert or evening snack)
```

### 4 Meals / Day (includes post-workout)
```
Breakfast:        25%
Lunch:            30%
Post-workout:     20%
Dinner:           25%
```

### 5–6 Meals / Day (bodybuilder-style)
```
Meal 1 (breakfast):    20%
Meal 2 (mid-morning):  15%
Meal 3 (lunch):        25%
Meal 4 (afternoon):    15%
Meal 5 (dinner):       20%
Meal 6 (optional):     5% (casein protein, cottage cheese)
```

---

## Dietary Restriction Substitutions

| Restriction | Avoid | Substitute |
|---|---|---|
| Gluten-free | Wheat, barley, rye, most bread/pasta | Rice, quinoa, gluten-free oats, potatoes |
| Dairy-free | Milk, cheese, yogurt, butter | Almond/oat/soy milk, coconut yogurt, olive oil |
| Vegan | All animal products | Legumes, tofu, tempeh, seitan (if no gluten issue) |
| Vegetarian | Meat, fish | Eggs, dairy, legumes, tofu |
| Nut allergy | All nuts + nut butters | Seeds (pumpkin, sunflower), tahini |
| Egg allergy | Eggs, mayo | Flax egg (1 tbsp ground flax + 3 tbsp water = 1 egg), chia egg |
| Low-FODMAP | Onion, garlic, legumes, lactose | Garlic-infused oil, rice, strawberries, hard cheese |

---

## Sample 7-Day Meal Plan Template (1800 kcal, balanced, no gluten)

> This is a template. Adjust portions to match user's calculated targets.

**Day 1**
- Breakfast: Greek yogurt (200g) + blueberries + honey + gluten-free granola
- Lunch: Chicken breast (150g) + quinoa (80g dry) + roasted vegetables + olive oil
- Dinner: Salmon (150g) + sweet potato (200g) + steamed broccoli
- Snack: Apple + almond butter (2 tbsp)
- ~Macros: 1,820 kcal | 145g P | 190g C | 52g F~

**Day 2**
- Breakfast: Omelette (3 eggs) + spinach + feta + tomato
- Lunch: Tuna (150g canned) + rice cakes + cucumber + hummus
- Dinner: Turkey mince stir-fry + rice noodles + mixed veg + tamari sauce
- Snack: Cottage cheese (150g) + pineapple
- ~Macros: ~1,780 kcal | 148g P | 185g C | 50g F~

**Day 3–7:** Rotate proteins (chicken / fish / eggs / legumes), vary vegetables, keep carbs from rice / potato / quinoa / oats.

---

## Grocery List Generation

When generating a meal plan, output a consolidated grocery list grouped by category:

```
### Proteins
- Chicken breast: X kg
- Salmon fillets: X portions
- Greek yogurt: X containers
- Eggs: X dozen

### Carbs / Grains
- Quinoa: X kg
- Basmati rice: X kg
- Sweet potato: X kg

### Vegetables
- Broccoli: X heads
- Spinach: X bags
- Mixed stir-fry veg: X bags

### Fruits
- Blueberries: X punnets
- Apples: X

### Fats / Condiments
- Olive oil: 1 bottle
- Almond butter: 1 jar
- Hummus: X tubs

### Other
- Gluten-free tamari: 1 bottle
- Honey: 1 jar
```

---

## Hydration Guidelines

| Body Weight | Minimum Daily Water |
|---|---|
| <60 kg | 1.8–2.2 L |
| 60–80 kg | 2.2–2.8 L |
| 80–100 kg | 2.8–3.2 L |
| >100 kg | 3.2–4.0 L |

Add 500 ml per 60 min of intense exercise. Increase in hot climates.

---

## Evidence-Based Supplements (Only These)

| Supplement | Evidence Level | Use Case |
|---|---|---|
| Creatine monohydrate | ⭐⭐⭐ Strong | Strength, power output, muscle gain |
| Protein powder (whey/plant) | ⭐⭐⭐ Strong | Hit protein targets when diet falls short |
| Vitamin D3 | ⭐⭐⭐ Strong | Deficiency common in low-sun climates |
| Omega-3 (fish oil) | ⭐⭐ Moderate | Inflammation, cardiovascular health |
| Magnesium glycinate | ⭐⭐ Moderate | Sleep quality, muscle recovery |
| Caffeine | ⭐⭐⭐ Strong | Performance, endurance |
| Beta-alanine | ⭐⭐ Moderate | Muscular endurance (>60s efforts) |

> ❌ **Do NOT recommend:** Detox products, proprietary "fat burners," hormone precursors, anything making disease claims.

---

## Sleep & Recovery Nutrition

- **Pre-sleep protein:** 30–40g casein (cottage cheese, Greek yogurt) supports overnight muscle protein synthesis
- **Post-workout window:** Aim for protein + carbs within 1–2 hours (strict timing is less critical than total daily intake)
- **Alcohol:** Disrupts sleep stages, suppresses protein synthesis. 2+ drinks = measurable recovery impact
- **Caffeine cutoff:** Avoid after 14:00 (half-life ~5 hours)
