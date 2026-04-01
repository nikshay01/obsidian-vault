# routes/workSessions.js — Work Session CRUD Routes

```js
const crudRouter = require('../utils/crudRouter');
const WorkSession = require('../models/WorkSession');

module.exports = crudRouter(WorkSession, { dateField: 'startTime' });
```
- Uses `startTime` as the date field (work sessions are filtered by when they started)
- Generates GET all, GET one, POST, PUT, DELETE endpoints via [[utils/crudRouter]]
- POST/PUT triggers pre-save hooks in [[models/WorkSession]] that auto-compute: `durationMin`, pause durations, `productiveTimeMin` (uses [[utils/computedFields]])
