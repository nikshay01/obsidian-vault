# routes/screenTime.js — Screen Time CRUD Routes

```js
const crudRouter = require('../utils/crudRouter');
const ScreenTime = require('../models/ScreenTime');
module.exports = crudRouter(ScreenTime, { dateField: 'date' });
```
- Uses `date` for filtering
- POST/PUT triggers pre-save that auto-computes `autopilotPercent` = `100 - intentionalPercent`
