# models/GamingSession.js — Gaming Session Schema

Tracks gaming sessions with auto-computed duration, match tracking, and custom metrics.

---

## Key Points

```js
    durationMinutes: { type: Number }, // computed
```
- Auto-calculated from `timestampStart` and `timestampEnd` in pre-save hook

```js
    matches: {
      played: { type: Number, default: 0 },
      won: { type: Number, default: 0 },
      lost: { type: Number, default: 0 },
    },
```
- `default: 0` — if not provided, starts at 0 (no null values)

```js
    customMetrics: [
      {
        metricName: { type: String },
        value: { type: String },
        intensityOrScale: { type: Number },
        notes: { type: String },
      },
    ],
```
- Flexible custom metrics array — track anything specific to a game

## Pre-Save Hook

```js
gamingSessionSchema.pre('save', function (next) {
  if (this.timestampStart && this.timestampEnd) {
    this.durationMinutes = durationMinutes(this.timestampStart, this.timestampEnd);
  }
  next();
});
```
- Auto-computes duration from start/end timestamps
- Same pattern used in WorkSession, HobbySession, and PainLog

```js
module.exports = mongoose.model('GamingSession', gamingSessionSchema);
```
- Creates `gamingsessions` collection
