# routes/dailySummary.js — Daily Summary CRUD Routes

```js
const crudRouter = require('../utils/crudRouter');
const DailySummary = require('../models/DailySummary');
module.exports = crudRouter(DailySummary, { dateField: 'date' });
```
- Uses `date` for filtering via [[utils/crudRouter]]
- Has a unique compound index in [[models/DailySummary]]: one summary per user per day (trying to create a second for the same day will error via [[middleware/errorHandler]])
