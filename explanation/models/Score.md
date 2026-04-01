# models/Score.js — Daily Composite Scores Schema

Stores daily aggregated scores across all modules with detailed breakdowns and trends. One score entry per user per day.

---

## Composite Scores (0-100)

```js
    dailyWellnessScore: { type: Number, min: 0, max: 100 },
    productivityIndex: { type: Number, min: 0, max: 100 },
    // ... spiritualScore, mentalHealthScore, physicalHealthScore, 
    //     sleepScore, screenDisciplineScore, hobbyGrowthScore, selfControlScore
    dayScore: { type: Number, min: 0, max: 100 }, // master score
    dayRating: { type: String, enum: ['Elite', 'Great', 'Good', 'Average', 'Rough', 'Bad'] },
```
- 10 composite scores, each 0-100
- `dayScore` — the master "how was your day" number
- `dayRating` — human-readable category derived from dayScore

## Breakdowns

Each composite score has a breakdown object showing what contributed to it:

```js
    wellnessBreakdown: {
      energyAvg: { type: Number },
      sleepQuality: { type: Number },
      moodAvg: { type: Number },
      negativeSymptomsInv: { type: Number },
      // ...
    },
```
- `Inv` suffix means "inverted" — lower pain/fatigue = higher score
- These breakdowns enable the frontend to show "why your wellness score is 72"

## Trends

```js
    trends: {
      meditationProgress: { type: Number },  // this week vs last week
      moodTrend: { type: Number },           // 7-day vs 30-day average
      sleepConsistency: { type: Number },     // std deviation of sleep times
      // ... more trend metrics
    },
```
- Computed over time windows (not in pre-save — would require aggregation pipeline)
- Application or scheduled job would compute these

## Metadata

```js
    computedAt: { type: Date },
    dataCompletenessPercent: { type: Number, min: 0, max: 100 },
    missingSources: [{ type: String }],
```
- `dataCompletenessPercent` — what % of the day's data was actually logged
- `missingSources: ["nutrition", "screentime"]` — which modules had no data

## Unique Constraint

```js
scoreSchema.index({ userId: 1, date: 1 }, { unique: true });
```
- One score per user per day — same pattern as DailySummary

```js
module.exports = mongoose.model('Score', scoreSchema);
```
