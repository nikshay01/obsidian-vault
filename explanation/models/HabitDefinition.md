# models/HabitDefinition.js — Habit Definition Schema

Defines a trackable habit with flexible completion types, frequency scheduling, streak tracking, and procrastination monitoring. Actual daily logs go in [[models/HabitLog]]. Exposed via [[routes/habits]].

---

## Completion System

```js
    completionType: {
      type: String,
      enum: ['boolean', 'duration', 'percentage', 'fraction', 'count', 'custom'],
      default: 'boolean',
    },
    targetValue: { type: Number, default: 1 },
    targetUnit: { type: String, enum: ['hours', 'minutes', 'pages', 'steps', '%', ''], default: '' },
```
- Versatile completion system:
  - `boolean` — did it or not
  - `duration` — logged hours/minutes (target: 3 hours)
  - `percentage` — logged as % (target: 100%)
  - `count` — logged as count (target: 10000 steps)
  - etc.

## Frequency

```js
    frequency: {
      type: { type: String, enum: ['daily', 'weekly', 'monthly', 'yearly', 'custom'] },
      timesPerPeriod: { type: Number, default: 1 },
      customDays: [{ type: String }],
      timeBlock: {
        enabled: { type: Boolean, default: false },
        startTime: { type: String }, // "05:00"
        endTime: { type: String },   // "06:00"
      },
    },
```
- `customDays: ["monday", "thursday"]` — for habits that aren't daily
- `timeBlock` — optional scheduled time slot (when enabled, powers procrastination tracking)

## Streak

```js
    streak: {
      current: { type: Number, default: 0 },
      longest: { type: Number, default: 0 },
      lastLoggedDate: { type: Date },
      allowedMissesBeforeBreak: { type: Number, default: 1 },
      currentConsecutiveMisses: { type: Number, default: 0 },
    },
```
- `allowedMissesBeforeBreak` — allows missing 1 day without breaking the streak (configurable)
- Application logic would update these fields when habit logs are created

## Procrastination

```js
    procrastination: {
      enabled: { type: Boolean, default: false },
      totalMissedPeriods: { type: Number, default: 0 },
      missedDates: [{ type: Date }],
      longestMissStreak: { type: Number, default: 0 },
      currentMissStreak: { type: Number, default: 0 },
    },
```
- `missedDates` — array of every missed period date (for trend analysis)
- `longestMissStreak` and `currentMissStreak` — calculated from missedDates

```js
module.exports = mongoose.model('HabitDefinition', habitDefinitionSchema);
```
