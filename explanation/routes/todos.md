# routes/todos.js — Todo CRUD Routes

```js
const crudRouter = require('../utils/crudRouter');
const Todo = require('../models/Todo');
module.exports = crudRouter(Todo, { dateField: 'createdAt' });
```
- Uses `createdAt` for date filtering via [[utils/crudRouter]]
- POST/PUT triggers pre-save in [[models/Todo]] that:
  - Auto-computes `actualDuration` = sum of all session durations using [[utils/computedFields]]
  - Auto-sets `completedAt` when status changes to `'completed'`
