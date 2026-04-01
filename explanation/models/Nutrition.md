# models/Nutrition.js — Nutrition Daily Log Schema

Tracks daily nutrition with meals, food items, and auto-computed totals at both meal and daily levels.

---

## Three-Level Structure

```
Nutrition Day
  └── meals[] (breakfast, lunch, dinner, etc.)
       └── items[] (each food item)
            └── nutrition (calories, protein, carbs, fat, fiber)
```

## Sub-Schemas

```js
const nutritionItemSchema = new mongoose.Schema({
  foodName: { type: String, required: true },
  portion: {
    quantity: { type: Number },
    unit: { type: String, enum: ['g', 'ml', 'piece', 'bowl', 'tsp', 'tbsp', 'cup', 'plate'] },
  },
  nutrition: {
    caloriesKcal: { type: Number },
    proteinG: { type: Number },
    // ... carbsG, fatG, fiberG
  },
}, { _id: false });
```
- Each food item has: name, portion size/unit, and nutritional breakdown
- `enum` on unit — only valid measurement units accepted

```js
const mealSchema = new mongoose.Schema({
  name: { type: String, enum: ['breakfast', 'lunch', 'snack', 'dinner', ...] },
  items: [nutritionItemSchema],
  mealTotals: { ... }, // computed
}, { _id: true });
```
- Meals DO get their own `_id` (`{ _id: true }`) — useful for referencing specific meals

---

## Pre-Save: Two-Level Aggregation

```js
nutritionSchema.pre('save', function (next) {
  const keys = ['caloriesKcal', 'proteinG', 'carbsG', 'fatG', 'fiberG'];
```
- Defines the 5 nutrient keys to sum across

### Level 1: Meal Totals

```js
  this.meals.forEach((meal) => {
    meal.mealTotals = {};
    keys.forEach((key) => {
      meal.mealTotals[key] = meal.items.reduce(
        (sum, item) => sum + ((item.nutrition && item.nutrition[key]) || 0), 0
      );
    });
  });
```
- For each meal → for each nutrient → sum all food items
- `item.nutrition && item.nutrition[key]` — safe property access (handles missing nutrition data)
- Example: breakfast has 2 eggs (140 cal) + toast (80 cal) → mealTotals.caloriesKcal = 220

### Level 2: Daily Totals

```js
  this.dailyTotals = {};
  keys.forEach((key) => {
    this.dailyTotals[key] = this.meals.reduce(
      (sum, meal) => sum + ((meal.mealTotals && meal.mealTotals[key]) || 0), 0
    );
  });
```
- Sums mealTotals across all meals for the day
- Example: breakfast (220 cal) + lunch (500 cal) + dinner (600 cal) → dailyTotals.caloriesKcal = 1320

```js
module.exports = mongoose.model('Nutrition', nutritionSchema);
```
- Creates `nutritions` collection
