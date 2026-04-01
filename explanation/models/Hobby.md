# models/Hobby.js — Hobby Definition Schema

Defines a hobby with goals, milestones, and aggregated stats. This is the "template" — actual sessions go in [[models/HobbySession]]. Exposed via [[routes/hobbies]].

---

## Milestone Sub-Schema

```js
const milestoneSchema = new mongoose.Schema({
  title: { type: String, required: true },
  achievedAt: { type: Date },
  note: { type: String },
}, { _id: true });
```
- `{ _id: true }` — milestones get their own IDs (for updating/deleting specific milestones)

## Stats

```js
    stats: {
      totalSessions: { type: Number, default: 0 },
      totalMinutes: { type: Number, default: 0 },
      avgSessionMinutes: { type: Number, default: 0 },
      longestSessionMinutes: { type: Number, default: 0 },
      currentStreakDays: { type: Number, default: 0 },
      longestStreakDays: { type: Number, default: 0 },
      lastSessionAt: { type: Date },
      avgQualityScore: { type: Number, default: 0 },
    },
```
- All default to 0 — these would be updated by application logic when hobby sessions are logged
- Could be auto-updated via `afterCreate` hook in the CRUD router in a future version

```js
module.exports = mongoose.model('Hobby', hobbySchema);
```
