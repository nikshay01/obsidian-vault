# routes/hobbies.js — Hobby Definition CRUD Routes

```js
const crudRouter = require('../utils/crudRouter');
const Hobby = require('../models/Hobby');
module.exports = crudRouter(Hobby, { dateField: 'createdAt' });
```
- Uses `createdAt` for date filtering (hobbies are definitions, not time-stamped entries)
- No computed fields — stats are updated by hobby session logging
