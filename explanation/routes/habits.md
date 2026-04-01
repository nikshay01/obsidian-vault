# routes/habits.js — Habit Definition CRUD Routes

```js
const crudRouter = require('../utils/crudRouter');
const HabitDefinition = require('../models/HabitDefinition');
module.exports = crudRouter(HabitDefinition, { dateField: 'createdAt' });
```
- Uses `createdAt` for date filtering
- Habit definitions are the "template" (name, frequency, target, etc.)
- Actual daily logging goes via the `/habit-logs` endpoint
