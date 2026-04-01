# models/MoodEntry.js — Mood Entry Schema

Tracks mood snapshots with 15+ mental/emotional metrics and multi-entry emotions.

---

## Emotion Sub-Schema

```js
const emotionSchema = new mongoose.Schema(
  {
    name: { type: String, required: true },
    intensity: { type: Number, min: 0, max: 10 },
  },
  { _id: false }
);
```
- Allows logging multiple emotions at once: `[{ name: "joy", intensity: 8 }, { name: "anxiety", intensity: 3 }]`
- Each emotion has a name and intensity scale

## Main Fields

- 15 metrics all following the same `{ type: Number, min: 0, max: 10 }` pattern
- Covers: mood, energy, mentalClarity, calmness, anxiety, tiredness, confidence, motivation, frustration, stress, overthinking, brainFog, sickness, laziness, boredom, mindfulness
- Context fields: activity, location, social situation, free-text note

No computed fields — all values are directly user-entered.

```js
module.exports = mongoose.model('MoodEntry', moodEntrySchema);
```
- Creates `moodentries` collection
