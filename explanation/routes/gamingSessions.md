# routes/gamingSessions.js — Gaming Session CRUD Routes

```js
const crudRouter = require('../utils/crudRouter');
const GamingSession = require('../models/GamingSession');
module.exports = crudRouter(GamingSession, { dateField: 'timestampStart' });
```
- Uses `timestampStart` for date filtering via [[utils/crudRouter]]
- POST/PUT triggers pre-save in [[models/GamingSession]] that auto-computes `durationMinutes` from `timestampStart` and `timestampEnd` using [[utils/computedFields]]
