# routes/sleep.js — Sleep CRUD Routes

```js
const crudRouter = require('../utils/crudRouter');
```
- Imports the CRUD factory function

```js
const Sleep = require('../models/Sleep');
```
- Imports the Sleep Mongoose model

```js
module.exports = crudRouter(Sleep, { dateField: 'date' });
```
- Calls the factory with the Sleep model
- `dateField: 'date'` — tells the factory to filter by the `date` field when using `?date=`, `?from=`, `?to=` query params
- This single line generates ALL 5 endpoints:
  - `GET /api/v1/sleep` — list all sleep records (paginated, user-scoped)
  - `GET /api/v1/sleep/:id` — get one sleep record
  - `POST /api/v1/sleep` — create a sleep record (triggers pre-save: computes totalSleepHours, deltas, etc.)
  - `PUT /api/v1/sleep/:id` — update a sleep record (re-triggers computed fields)
  - `DELETE /api/v1/sleep/:id` — delete a sleep record

All routes are protected by the `protect` middleware applied in `server.js`.
