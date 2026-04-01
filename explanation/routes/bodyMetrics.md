# routes/bodyMetrics.js — Body Metric CRUD Routes

```js
const crudRouter = require('../utils/crudRouter');
const BodyMetric = require('../models/BodyMetric');
module.exports = crudRouter(BodyMetric, { dateField: 'recordedAt' });
```
- Uses `recordedAt` for date filtering via [[utils/crudRouter]]
- No computed fields — all measurements in [[models/BodyMetric]] are directly entered
