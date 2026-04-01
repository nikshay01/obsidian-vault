# models/Meditation.js — Meditation Session Schema

Tracks meditation sessions with pre/post mental states, practice metrics, physiological response, and spiritual state.

---

## Key Fields

```js
    timestampStart: { type: Date, required: true },
    durationMinutes: { type: Number, required: true },
```
- Both required — every meditation session must have a start time and duration

```js
    type: {
      type: String,
      enum: [
        'breath', 'mantra', 'guided', 'silent', 'walking', 'body_scan',
        'mindfulness', 'spiritual', 'focused', 'movement', 'transcendental',
        'progressive_relaxation', 'loving_kindness', 'visualization',
      ],
    },
```
- 14 meditation types supported
- `enum` validates that only these values are accepted

## State Sections

```js
    preState: {
      stressLevel: { type: Number, min: 0, max: 10 },
      energyLevel: { type: Number, min: 0, max: 10 },
      // ... 4 more fields
    },
    postState: {
      stressLevel: { type: Number, min: 0, max: 10 },
      // ... 5 more fields
    },
```
- Pre and post meditation mental state — used to measure meditation's effect
- All fields are 0-10 scale

```js
    practiceMetrics: {
      depthLevel: { type: Number, min: 0, max: 10 },
      focusPercentage: { type: Number, min: 0, max: 100 },
      mantraCount: { type: Number },
    },
```
- `focusPercentage` has a wider range (0-100) unlike other 0-10 fields
- `mantraCount` — no min/max, just a positive integer

```js
    spiritualState: {
      senseOfPresence: { type: Number, min: 0, max: 10 },
      devotionalFeeling: { type: Number, min: 0, max: 10 },
      gratitudeLevel: { type: Number, min: 0, max: 10 },
    },
```
- Spiritual depth metrics specific to devotional meditation

---

No pre-save hooks — all values are directly user-entered. The session quality score could be auto-computed in a future version.

```js
module.exports = mongoose.model('Meditation', meditationSchema);
```
- Creates `meditations` collection
