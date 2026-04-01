# models/BodyMetric.js — Body Measurement Schema

Tracks physical body measurements organized by body region.

---

## Structure

```js
    measurements: {
      arms: { leftBicepCm, rightBicepCm, leftForearmCm, rightForearmCm, ... },
      torso: { chestRelaxedCm, chestFlexedCm, waistCm, abdominalCm, hipsCm },
      shoulders: { leftCm, rightCm, widthCm },
      legs: { leftThighCm, rightThighCm, leftCalfCm, rightCalfCm },
      body: { heightCm, weightKg, neckCm, wristCm, bodyFatPercent },
    },
```
- Deeply nested object organized by body region
- All measurements are `{ type: Number }` — no min/max constraints (measurements vary widely)
- `bodyFatPercent` — the most commonly tracked metric for body composition

```js
    recordedAt: { type: Date, default: Date.now, index: true },
```
- `default: Date.now` — auto-sets to current time if not provided
- Note: `Date.now` (without parentheses) — Mongoose calls it at document creation time (not at schema definition time)

No computed fields.

```js
module.exports = mongoose.model('BodyMetric', bodyMetricSchema);
```
