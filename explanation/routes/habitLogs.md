# routes/habitLogs.js — Habit Log CRUD Routes

```js
const crudRouter = require('../utils/crudRouter');
const HabitLog = require('../models/HabitLog');
module.exports = crudRouter(HabitLog, { dateField: 'periodDate' });
```
- Uses `periodDate` for date filtering (the date/period this log belongs to) via [[utils/crudRouter]]
- POST/PUT triggers pre-save in [[models/HabitLog]] that auto-computes `completionPercent` = `(periodTotalValue / targetValue) * 100` using [[utils/computedFields]]
