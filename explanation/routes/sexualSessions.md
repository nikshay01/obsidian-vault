# routes/sexualSessions.js — Sexual Session CRUD Routes

```js
const crudRouter = require('../utils/crudRouter');
const SexualSession = require('../models/SexualSession');
module.exports = crudRouter(SexualSession, { dateField: 'timestampStart' });
```
- Uses `timestampStart` for date filtering via [[utils/crudRouter]]
- No auto-computed fields — all values in [[models/SexualSession]] are directly entered
