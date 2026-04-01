# models/Devotion.js — Devotion Session Schema

Tracks spiritual devotion sessions with cultural/Hindu-specific fields and auto-computed stress delta. Exposed via [[routes/devotion]].

---

## Key Fields

```js
    devotionType: {
      type: String,
      enum: ['mandir', 'puja', 'aarti', 'jaap', 'path', 'kirtan', 'seva', 'satsang', 'vrat', 'other'],
    },
    deityFocus: {
      type: String,
      enum: ['Shiva', 'Vishnu', 'Devi', 'Ganesha', 'Ram', 'Krishna', 'other'],
    },
```
- Culturally specific enums for devotion types and deity focus
- Keeps data consistent and queryable

```js
    customFields: [
      {
        name: { type: String },
        value: { type: mongoose.Schema.Types.Mixed },
      },
    ],
```
- `Schema.Types.Mixed` — accepts ANY value type (string, number, boolean, object, etc.)
- Used for custom per-user metrics like "Ram name chants: 108"

---

## Pre-Save Hook

```js
devotionSchema.pre('save', function (next) {
  if (this.preStressLevel != null && this.postStressLevel != null) {
    this.stressDelta = this.preStressLevel - this.postStressLevel;
  }
  next();
});
```
- `stressDelta` = pre - post stress
- Positive delta = stress REDUCED (good devotion session)
- Negative delta = stress INCREASED (unusual)
- `!= null` — checks for both null and undefined but NOT 0 (0 is a valid stress level)

```js
module.exports = mongoose.model('Devotion', devotionSchema);
```
- Creates `devotions` collection
