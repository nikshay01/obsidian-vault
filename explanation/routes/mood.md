# routes/mood.js — Mood Entry CRUD Routes

```js
const crudRouter = require('../utils/crudRouter');
const MoodEntry = require('../models/MoodEntry');
module.exports = crudRouter(MoodEntry, { dateField: 'timestamp' });
```
- Uses `timestamp` for date filtering
- No computed fields — all values are directly user-entered
