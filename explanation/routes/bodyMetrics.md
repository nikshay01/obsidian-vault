# routes/bodyMetrics.js — Body Metric CRUD Routes

```js
const crudRouter = require('../utils/crudRouter');
const BodyMetric = require('../models/BodyMetric');
module.exports = crudRouter(BodyMetric, { dateField: 'recordedAt' });
```
- Uses `recordedAt` for date filtering
- No computed fields — all measurements are directly entered
