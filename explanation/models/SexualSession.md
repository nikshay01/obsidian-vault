# models/SexualSession.js — Sexual Session Schema

Tracks sexual health sessions with pre/post mental states, trigger analysis, and self-regulation metrics. Exposed via [[routes/sexualSessions]].

---

## Key Fields

```js
    sessionType: {
      type: String,
      enum: ['masturbation', 'partnered sex', 'mutual masturbation', 'oral', 'other'],
    },
    triggerType: {
      type: String,
      enum: ['habit', 'boredom', 'sexual desire', 'partner interaction'],
    },
```
- Enums keep data consistent for trend analysis

```js
    preState: {
      mood: { type: Number, min: 0, max: 10 },
      energy: { type: Number, min: 0, max: 10 },
      stress: { type: Number, min: 0, max: 10 },
      anxiety: { type: Number, min: 0, max: 10 },
      guilt: { type: Number, min: 0, max: 10 },
      brainFog: { type: Number, min: 0, max: 10 },
      mentalClarity: { type: Number, min: 0, max: 10 },
    },
    postState: { ... }, // same 7 fields
```
- Pre and post mental state comparison — allows tracking psychological impact

```js
    urgeResistanceAttempts: { type: Number },
    regretLevel: { type: Number, min: 0, max: 10 },
```
- Self-regulation tracking metrics

No computed fields — all values are directly entered.

```js
module.exports = mongoose.model('SexualSession', sexualSessionSchema);
```
