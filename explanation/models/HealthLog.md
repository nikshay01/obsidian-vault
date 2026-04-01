# models/HealthLog.js — Health Log Schema

Tracks daily health metrics: energy throughout the day, symptoms, and physical state. Exposed via [[routes/healthLogs]].

---

## Key Fields

```js
    morningEnergy: { type: Number, min: 0, max: 10 },
    middayEnergy: { type: Number, min: 0, max: 10 },
    eveningEnergy: { type: Number, min: 0, max: 10 },
```
- Three-point energy readings throughout the day — useful for tracking energy patterns

```js
    sickStatus: { type: Boolean },
    sicknessIntensity: { type: Number, min: 0, max: 10 },
    symptoms: [{ type: String }],
```
- `sickStatus` — boolean flag (sick or not)
- `symptoms` — array of strings: `["headache", "sore throat", "fatigue"]`

```js
    restingHeartRate: { type: Number },
```
- No min/max constraint — heart rates vary widely (40-100+ BPM is normal range)

No computed fields — all directly entered.

```js
module.exports = mongoose.model('HealthLog', healthLogSchema);
```
