# models/PainLog.js — Pain Log Schema

Tracks pain episodes with per-body-part intensities and auto-computed duration. Uses [[utils/computedFields]]. Exposed via [[routes/painLogs]].

---

## Key Fields

```js
    bodyParts: [{ type: String }],
```
- Array of strings — can log multiple body parts affected (e.g., `["lower back", "left shoulder"]`)

```js
    bodyPartIntensities: [
      {
        bodyPart: { type: String },
        intensity: { type: Number, min: 0, max: 10 },
      },
    ],
```
- Optional per-body-part intensity tracking (more granular than the single `intensity` field)

```js
    postPain: {
      painEndTimestamp: { type: Date },
      totalDurationMinutes: { type: Number },
      avgIntensity: { type: Number, min: 0, max: 10 },
    },
```
- Post-pain follow-up tracking — how long the pain lasted total and average intensity throughout

## Pre-Save

```js
painLogSchema.pre('save', function (next) {
  if (this.painStartTimestamp && this.painEndTimestamp) {
    this.durationMinutes = durationMinutes(this.painStartTimestamp, this.painEndTimestamp);
  }
  next();
});
```
- Auto-computes duration from start/end timestamps using [[utils/computedFields|durationMinutes()]]

```js
module.exports = mongoose.model('PainLog', painLogSchema);
```
