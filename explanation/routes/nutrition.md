# routes/nutrition.js — Nutrition CRUD Routes

```js
const crudRouter = require('../utils/crudRouter');
const Nutrition = require('../models/Nutrition');
module.exports = crudRouter(Nutrition, { dateField: 'date' });
```
- Uses `date` for filtering
- POST/PUT triggers pre-save that auto-computes `mealTotals` (per-meal sum) and `dailyTotals` (whole-day sum) for calories, protein, carbs, fat, fiber
