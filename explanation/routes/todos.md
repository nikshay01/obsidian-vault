# routes/todos.js — Todo CRUD Routes

```js
const crudRouter = require('../utils/crudRouter');
const Todo = require('../models/Todo');
module.exports = crudRouter(Todo, { dateField: 'createdAt' });
```
- Uses `createdAt` for date filtering
- POST/PUT triggers pre-save that:
  - Auto-computes `actualDuration` = sum of all session durations
  - Auto-sets `completedAt` when status changes to `'completed'`
