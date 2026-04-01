# models/HobbySession.js — Hobby Session Schema

Tracks individual hobby practice sessions with quality metrics and auto-computed duration.

---

## Key Fields

```js
    hobbyId: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'Hobby',
      required: true,
      index: true,
    },
```
- References which Hobby this session belongs to
- `ref: 'Hobby'` — enables `.populate('hobbyId')` to load the full hobby data
- Indexed for fast lookup of all sessions for a specific hobby

```js
    attachments: [
      {
        type: { type: String, enum: ['text', 'photo', 'video'] },
        url: { type: String },
        description: { type: String },
      },
    ],
```
- Supports multimedia attachments (photos of artwork, video of practice, etc.)

## Pre-Save

```js
hobbySessionSchema.pre('save', function (next) {
  if (this.startedAt && this.endedAt) {
    this.durationMinutes = durationMinutes(this.startedAt, this.endedAt);
  }
  next();
});
```
- Auto-computes duration from start/end — same pattern as other session models

```js
module.exports = mongoose.model('HobbySession', hobbySessionSchema);
```
