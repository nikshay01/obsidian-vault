# routes/painLogs.js — Pain Log CRUD Routes

```js
const crudRouter = require('../utils/crudRouter');
const PainLog = require('../models/PainLog');
module.exports = crudRouter(PainLog, { dateField: 'timestamp' });
```
- Uses `timestamp` for date filtering via [[utils/crudRouter]]
- POST/PUT triggers pre-save in [[models/PainLog]] that auto-computes `durationMinutes` from `painStartTimestamp` and `painEndTimestamp` using [[utils/computedFields]]
