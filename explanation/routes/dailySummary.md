# routes/dailySummary.js — Daily Summary CRUD Routes

```js
const crudRouter = require('../utils/crudRouter');
const DailySummary = require('../models/DailySummary');
module.exports = crudRouter(DailySummary, { dateField: 'date' });
```
- Uses `date` for filtering
- Has a unique compound index: one summary per user per day (trying to create a second for the same day will error)
