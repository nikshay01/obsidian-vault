# routes/sexualSessions.js — Sexual Session CRUD Routes

```js
const crudRouter = require('../utils/crudRouter');
const SexualSession = require('../models/SexualSession');
module.exports = crudRouter(SexualSession, { dateField: 'timestampStart' });
```
- Uses `timestampStart` for date filtering
- No auto-computed fields — all values are directly entered
