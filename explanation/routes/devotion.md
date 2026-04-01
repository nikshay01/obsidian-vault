# routes/devotion.js — Devotion CRUD Routes

```js
const crudRouter = require('../utils/crudRouter');
const Devotion = require('../models/Devotion');
module.exports = crudRouter(Devotion, { dateField: 'timestampStart' });
```
- Uses `timestampStart` for date filtering via [[utils/crudRouter]]
- POST/PUT triggers pre-save in [[models/Devotion]] that auto-computes `stressDelta` (preStressLevel - postStressLevel) using [[utils/computedFields]]
