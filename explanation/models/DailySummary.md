# models/DailySummary.js — Daily Summary Schema

One summary per user per day — tracks overall day highlights, averages, and custom metrics. Exposed via [[routes/dailySummary]]. Used by [[models/Score]].

---

## Key Fields

```js
    overallDayRating: { type: Number, min: 0, max: 10 },
    highlights: { type: String, maxlength: 5000 },
    lowlights: { type: String, maxlength: 5000 },
    majorAchievements: [{ type: String }],
```
- `maxlength: 5000` — allows long-form text but prevents abuse
- `majorAchievements` — array of strings, no limit on count

## Unique Constraint

```js
dailySummarySchema.index({ userId: 1, date: 1 }, { unique: true });
```
- **Compound unique index** on `userId + date`
- `1` means ascending order
- This ensures only ONE summary per user per day
- Trying to create a second summary for the same day → MongoDB duplicate key error → caught by [[middleware/errorHandler|errorHandler.js]]

```js
module.exports = mongoose.model('DailySummary', dailySummarySchema);
```
