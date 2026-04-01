# routes/scores.js — Score CRUD Routes

```js
const crudRouter = require('../utils/crudRouter');
const Score = require('../models/Score');
module.exports = crudRouter(Score, { dateField: 'date' });
```
- Uses `date` for filtering
- Has a unique compound index: one score per user per day
- Stores all daily composite scores (wellness, productivity, spiritual, etc.) and their breakdowns
