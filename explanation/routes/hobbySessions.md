# routes/hobbySessions.js — Hobby Session CRUD Routes

```js
const crudRouter = require('../utils/crudRouter');
const HobbySession = require('../models/HobbySession');
module.exports = crudRouter(HobbySession, { dateField: 'startedAt' });
```
- Uses `startedAt` for date filtering via [[utils/crudRouter]]
- POST/PUT triggers pre-save in [[models/HobbySession]] that auto-computes `durationMinutes` from `startedAt` and `endedAt` using [[utils/computedFields]]
