# routes/hobbies.js — Hobby Definition CRUD Routes

```js
const crudRouter = require('../utils/crudRouter');
const Hobby = require('../models/Hobby');
module.exports = crudRouter(Hobby, { dateField: 'createdAt' });
```
- Uses `createdAt` for date filtering (hobbies are definitions, not time-stamped entries) via [[utils/crudRouter]]
- No computed fields — stats in [[models/Hobby]] are updated by hobby session logging (see [[routes/hobbySessions]])
