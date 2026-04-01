# models/HabitLog.js — Habit Log Schema

Individual log entries for habits — tracks what was logged, running totals, and auto-computed completion percentage. Belongs to a [[models/HabitDefinition]]. Uses [[utils/computedFields]]. Exposed via [[routes/habitLogs]].

---

## Key Fields

```js
    habitId: { type: mongoose.Schema.Types.ObjectId, ref: 'HabitDefinition', required: true },
    periodDate: { type: Date, required: true, index: true },
    loggedAt: { type: Date, default: Date.now },
```
- `habitId` — links to which [[models/HabitDefinition|habit definition]] this log is for
- `periodDate` — which day/week this log belongs to (normalized date)
- `loggedAt` — exact timestamp of when the entry was made

```js
    loggedValue: { type: Number },       // what you logged THIS entry (e.g., 1hr)
    periodTotalValue: { type: Number },  // running sum for the period (e.g., 3hr after 3 entries)
    completionPercent: { type: Number }, // computed
```
- Multiple entries per period are supported (log 1hr in morning, 1hr in evening = 2hr total)
- `periodTotalValue` should be maintained by the frontend/application logic

## Time Block Tracking

```js
    timeBlock: {
      planned: { start: { type: String }, end: { type: String } },
      actual: { start: { type: Date }, end: { type: Date } },
      minutesLate: { type: Number },
    },
```
- If the habit has a time block (e.g., 5:00-6:00 AM), tracks planned vs actual times
- `minutesLate` — how many minutes late you started

## Pre-Save

```js
habitLogSchema.pre('save', function (next) {
  if (this.periodTotalValue != null && this.targetValue != null) {
    this.completionPercent = completionPercent(this.periodTotalValue, this.targetValue);
  }
  next();
});
```
- Auto-computes: `(periodTotalValue / targetValue) * 100` using [[utils/computedFields|completionPercent()]]
- Example: logged 2 of 3 target hours → `66.7%`

```js
module.exports = mongoose.model('HabitLog', habitLogSchema);
```
