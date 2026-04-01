# routes/meditation.js — Meditation CRUD Routes

```js
const crudRouter = require('../utils/crudRouter');
const Meditation = require('../models/Meditation');
module.exports = crudRouter(Meditation, { dateField: 'timestampStart' });
```
- Uses `timestampStart` for date filtering
- Generates all 5 CRUD endpoints for meditation sessions
