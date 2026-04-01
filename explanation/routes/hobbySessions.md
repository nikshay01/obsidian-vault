# routes/hobbySessions.js — Hobby Session CRUD Routes

```js
const crudRouter = require('../utils/crudRouter');
const HobbySession = require('../models/HobbySession');
module.exports = crudRouter(HobbySession, { dateField: 'startedAt' });
```
- Uses `startedAt` for date filtering
- POST/PUT triggers pre-save that auto-computes `durationMinutes` from `startedAt` and `endedAt`
