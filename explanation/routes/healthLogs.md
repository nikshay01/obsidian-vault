# routes/healthLogs.js — Health Log CRUD Routes

```js
const crudRouter = require('../utils/crudRouter');
const HealthLog = require('../models/HealthLog');
module.exports = crudRouter(HealthLog, { dateField: 'date' });
```
- Uses `date` for filtering
- No computed fields — all health metrics are directly entered
