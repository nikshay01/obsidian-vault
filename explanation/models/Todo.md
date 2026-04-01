# models/Todo.js — Todo Schema

Full task management with deadlines, procrastination tracking, work sessions, and auto-computed durations. Exposed via [[routes/todos]].

---

## Sub-Schemas

```js
const checkpointSchema = new mongoose.Schema({
  recordedAt: { type: Date, default: Date.now },
  minutesSoFar: { type: Number },
}, { _id: false });
```
- Procrastination checkpoints — snapshots of how long you've been procrastinating
- Recorded on app open/close or manually

```js
const sessionSchema = new mongoose.Schema({
  start: { type: Date },
  end: { type: Date },
  duration: { type: Number }, // minutes
}, { _id: false });
```
- Work sessions on this task — a task can have multiple work sessions

## Deadline System

```js
    deadline: {
      type: { type: String, enum: ['exact', 'within', 'range', 'anytime'] },
      exactDate: { type: Date },       // for "exact"
      windowStart: { type: Date },      // for "range" / "within"
      windowEnd: { type: Date },        // for "range" / "within"
    },
```
- Flexible deadline types:
  - `exact` — must be done by exactDate
  - `within` — do it within this window
  - `range` — can be done anytime in this range
  - `anytime` — no deadline pressure

## Procrastination System

```js
    procrastination: {
      isActive: { type: Boolean, default: false },
      triggeredBy: { type: String, enum: ['auto', 'manual'] },
      clockStartedAt: { type: Date },
      totalMinutes: { type: Number, default: 0 },
      manuallyFlagged: { type: Boolean, default: false },
      checkpoints: [checkpointSchema],
    },
```
- `isActive` — true means procrastination clock is currently running
- `triggeredBy: 'auto'` — started automatically when estimatedStartTime passed
- `totalMinutes` — accumulates until task completion, then freezes
- `checkpoints` — history of procrastination progress

## Pre-Save

```js
todoSchema.pre('save', function (next) {
  if (this.sessions && this.sessions.length > 0) {
    this.actualDuration = this.sessions.reduce(
      (sum, s) => sum + (s.duration || 0), 0
    );
  }
```
- Sums all work session durations into `actualDuration`

```js
  if (this.status === 'completed' && !this.completedAt) {
    this.completedAt = new Date();
  }
  next();
});
```
- Auto-sets `completedAt` timestamp when status changes to completed
- `!this.completedAt` — only sets it once (doesn't change on subsequent saves)

```js
module.exports = mongoose.model('Todo', todoSchema);
```
