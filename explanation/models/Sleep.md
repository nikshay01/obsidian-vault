# models/Sleep.js — Sleep Log Schema

Handles sleep tracking with auto-computed durations, interruption totals, deltas, and nap/dream tracking.

---

## Sub-Schemas

```js
const interruptionSchema = new mongoose.Schema(
  {
    wakeTime: { type: Date },
    backToSleepTime: { type: Date },
    durationMinutes: { type: Number },
    reason: { type: String, enum: ['bathroom', 'noise', 'anxiety', 'pain', 'random', 'other'] },
  },
  { _id: false }
);
```
- Defines the shape of each sleep interruption entry
- `enum` — restricts `reason` to only these values (validation fails if anything else is passed)
- `{ _id: false }` — sub-documents don't get their own `_id` (saves space, these are embedded)

```js
const napSchema = new mongoose.Schema({ ... }, { _id: false });
const dreamSchema = new mongoose.Schema({ ... }, { _id: false });
```
- Same pattern: define the shape of array items with validation
- Naps: start/end time, duration, quality (0-10)
- Dreams: description, vividness (0-10), emotional tone (enum), lucid (boolean)

---

## Main Schema Fields

```js
    userId: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'User',
      required: true,
      index: true,
    },
```
- `ref: 'User'` — references the User collection (for Mongoose `.populate()`)
- `required: true` — every sleep record must belong to a user
- `index: true` — creates a database index for fast queries (filtering by userId is very common)

```js
    date: { type: Date, required: true, index: true },
```
- The date of the sleep entry (also indexed for fast date queries)

```js
    sleepStart: { type: Date },
    sleepEnd: { type: Date },
    totalSleepHours: { type: Number }, // computed
```
- Start/end are user-entered; totalSleepHours is auto-computed by pre-save

```js
    sleepQuality: { type: Number, min: 0, max: 10 },
```
- `min: 0, max: 10` — Mongoose validation: rejects values outside this range

```js
    interruptions: [interruptionSchema],
```
- Array of interruption sub-documents using the schema defined above

---

## Pre-Save Computations

```js
sleepSchema.pre('save', function (next) {
```
- Runs before every `.save()` — triggered by both POST (create) and PUT (update via save)

### Interruption Durations

```js
  if (this.interruptions && this.interruptions.length > 0) {
    this.interruptions.forEach((i) => {
      if (i.wakeTime && i.backToSleepTime && !i.durationMinutes) {
        i.durationMinutes = durationMinutes(i.wakeTime, i.backToSleepTime);
      }
    });
```
- For each interruption: if it has start/end times but no duration, compute it
- `!i.durationMinutes` — respects manually entered durations (won't overwrite)

```js
    this.totalInterruptionMinutes = this.interruptions.reduce(
      (sum, i) => sum + (i.durationMinutes || 0), 0
    );
    this.sleepInterruptionsCount = this.interruptions.length;
  }
```
- `.reduce()` — sums all interruption durations into a single number
- `(i.durationMinutes || 0)` — treats null/undefined as 0
- Also sets the count from the array length

### Total Sleep Hours

```js
  if (this.sleepStart && this.sleepEnd) {
    const rawHours = durationHours(this.sleepStart, this.sleepEnd);
    this.totalSleepHours = parseFloat(
      (rawHours - (this.totalInterruptionMinutes || 0) / 60).toFixed(2)
    );
  }
```
- `rawHours` — total time from going to sleep to waking up
- Subtracts interruption time (converted from minutes to hours: `/ 60`)
- `.toFixed(2)` — keeps 2 decimal places, `parseFloat` removes trailing zeros

### Sleep Hours Delta

```js
  if (this.totalSleepHours != null && this.targetSleepHours != null) {
    this.sleepHoursDelta = parseFloat(
      (this.totalSleepHours - this.targetSleepHours).toFixed(2)
    );
  }
```
- Difference between actual and target sleep
- Positive = overslept, Negative = underslept
- Example: slept 7, target 8 → delta = -1

### Dream Count

```js
  if (this.dreams) {
    this.dreamCount = this.dreams.length;
  }
  next();
});
```
- Auto-counts dreams from the array length

---

```js
module.exports = mongoose.model('Sleep', sleepSchema);
```
- Creates `sleeps` collection in MongoDB
