# models/ScreenTime.js — Screen Time Schema

Tracks daily screen time usage with auto-computed autopilot percentage.

---

## Key Fields

```js
    totalScreenTimeMinutes: { type: Number },
    unlockCount: { type: Number },
```
- Overall daily metrics

```js
    socialMediaMinutes: { type: Number },
    entertainmentMinutes: { type: Number },
    productiveMinutes: { type: Number },
    learningMinutes: { type: Number },
    gamingMinutes: { type: Number },
```
- Breakdown by category

```js
    shortFormUsed: { type: Boolean },
    lateNightUsage: { type: Boolean },
    inBedUsage: { type: Boolean },
```
- Boolean flags for negative screen time habits

```js
    intentionalPercent: { type: Number, min: 0, max: 100 },
    autopilotPercent: { type: Number, min: 0, max: 100 }, // computed
```
- `intentionalPercent` — user-entered: how much screen time was deliberate
- `autopilotPercent` — auto-computed in pre-save

## Pre-Save

```js
screenTimeSchema.pre('save', function (next) {
  if (this.intentionalPercent != null) {
    this.autopilotPercent = autopilotPercent(this.intentionalPercent);
  }
  next();
});
```
- `autopilotPercent(70)` returns `30` — the mindless usage percentage
- Uses the helper from `utils/computedFields.js` which clamps between 0-100

```js
module.exports = mongoose.model('ScreenTime', screenTimeSchema);
```
