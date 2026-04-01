# routes/scores.js — Score CRUD Routes

```js
const crudRouter = require('../utils/crudRouter');
const Score = require('../models/Score');
module.exports = crudRouter(Score, { dateField: 'date' });
```
- Uses `date` for filtering via [[utils/crudRouter]]
- Has a unique compound index in [[models/Score]]: one score per user per day (handled by [[middleware/errorHandler]])
- Stores all daily composite scores (wellness, productivity, spiritual, etc.) and their breakdowns
