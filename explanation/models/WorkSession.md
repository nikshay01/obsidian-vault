# models/WorkSession.js — Work Session Schema

Tracks work/study sessions with auto-computed duration, pause totals, and productive time.

---

## Pause Sub-Schema

```js
const pauseSchema = new mongoose.Schema(
  {
    start: { type: Date },
    end: { type: Date },
    durationMinutes: { type: Number },
    reason: { type: String },
  },
  { _id: false }
);
```
- Each pause has start/end time, computed duration, and optional reason
- `{ _id: false }` — no separate ID for embedded pause items

---

## Main Fields

```js
    startTime: { type: Date, required: true },
    endTime: { type: Date, required: true },
    durationMin: { type: Number }, // computed
    productiveTimeMin: { type: Number }, // computed
```
- `durationMin` — auto-computed: endTime - startTime
- `productiveTimeMin` — auto-computed: duration - total pause time

```js
    taskType: {
      type: String,
      enum: ['professional', 'personal', 'learning', 'creative'],
      required: true,
    },
```
- `enum` — only these 4 values are allowed; anything else fails validation

```js
    mentalState: {
      mentalEnergy: { type: Number, min: 0, max: 10 },
      // ... 12 more mental state fields, all 0-10
    },
```
- Nested object with 13 mental state metrics, all using the same `min: 0, max: 10` validation

---

## Pre-Save Hook

```js
workSessionSchema.pre('save', function (next) {
  if (this.startTime && this.endTime) {
    this.durationMin = durationMinutes(this.startTime, this.endTime);
  }
```
- Computes total session duration from start/end times

```js
  let totalPause = 0;
  if (this.pauses && this.pauses.length > 0) {
    this.pauses.forEach((p) => {
      if (p.start && p.end && !p.durationMinutes) {
        p.durationMinutes = durationMinutes(p.start, p.end);
      }
      totalPause += p.durationMinutes || 0;
    });
  }
```
- For each pause: compute its duration if not already set
- Accumulates total pause time

```js
  if (this.durationMin != null) {
    this.productiveTimeMin = Math.max(0, this.durationMin - totalPause);
  }
```
- Productive time = total duration minus pauses
- `Math.max(0, ...)` — never goes negative (even if pauses somehow exceed duration)

---

```js
module.exports = mongoose.model('WorkSession', workSessionSchema);
```
- Creates `worksessions` collection in MongoDB
