# routes/mood.js — Mood Entry CRUD Routes

```js
const crudRouter = require('../utils/crudRouter');
const MoodEntry = require('../models/MoodEntry');
module.exports = crudRouter(MoodEntry, { dateField: 'timestamp' });
```
- Uses `timestamp` for date filtering via [[utils/crudRouter]]
- No computed fields — all values in [[models/MoodEntry]] are directly user-entered
