# utils/crudRouter.js — Reusable CRUD Route Factory

This is the **most important architectural piece** in the backend. Instead of writing GET/POST/PUT/DELETE handlers 18 times, this factory generates them for ANY Mongoose model.

---

```js
const express = require('express');
```

```js
function crudRouter(Model, opts = {}) {
```
- **Factory function** — takes a Mongoose Model class and options, returns an Express Router
- `opts = {}` — defaults to empty object if no options passed

```js
  const router = express.Router();
  const dateField = opts.dateField || 'date';
```
- Creates a new Express router instance
- `dateField` — which field to use for date filtering (varies by module):
  - Sleep uses `'date'`
  - Work sessions use `'startTime'`
  - Mood entries use `'timestamp'`
  - etc.

---

## GET ALL — `GET /`

```js
  router.get('/', async (req, res, next) => {
    try {
      const filter = { userId: req.user._id };
```
- **Always** filters by the logged-in user's ID
- This is the core security mechanism — users can ONLY see their own data
- `req.user._id` was set by the `protect` middleware

### Date Filtering

```js
      if (req.query.date) {
        const d = new Date(req.query.date);
        const next_d = new Date(d);
        next_d.setDate(next_d.getDate() + 1);
        filter[dateField] = { $gte: d, $lt: next_d };
      }
```
- `?date=2026-03-24` → finds all records from that entire day
- Creates a range: `>= 2026-03-24T00:00:00Z` AND `< 2026-03-25T00:00:00Z`
- `$gte` = greater than or equal, `$lt` = less than
- This is timezone-safe — covers the full 24-hour period

```js
      else if (req.query.from || req.query.to) {
        filter[dateField] = {};
        if (req.query.from) filter[dateField].$gte = new Date(req.query.from);
        if (req.query.to) filter[dateField].$lte = new Date(req.query.to);
      }
```
- `?from=2026-03-01&to=2026-03-31` → date range query
- Either `from` or `to` can be used alone
- `$lte` = less than or equal (inclusive end date)

### Pagination

```js
      const page = Math.max(1, parseInt(req.query.page) || 1);
```
- `parseInt(req.query.page)` — converts query string to integer
- `|| 1` — defaults to page 1 if not provided or NaN
- `Math.max(1, ...)` — ensures page is at least 1 (no negative pages)

```js
      const limit = Math.min(100, Math.max(1, parseInt(req.query.limit) || 20));
```
- Defaults to 20 per page
- `Math.max(1, ...)` — at least 1 result
- `Math.min(100, ...)` — at most 100 results (prevents `?limit=999999` abuse)

```js
      const skip = (page - 1) * limit;
```
- Skip calculation: page 1 → skip 0, page 2 → skip 20, page 3 → skip 40

### Sorting

```js
      const sort = req.query.sort || `-${dateField}`;
```
- `?sort=-createdAt` → sort by createdAt descending (newest first)
- `-` prefix means descending in Mongoose
- Default: newest records first (descending by date field)

### Execute Query

```js
      const [docs, total] = await Promise.all([
        Model.find(filter).sort(sort).skip(skip).limit(limit).lean(),
        Model.countDocuments(filter),
      ]);
```
- `Promise.all()` — runs BOTH queries **in parallel** (faster than sequential)
- First query: finds matching documents with sorting, pagination
  - `.lean()` — returns plain JS objects instead of Mongoose documents (2-3x faster, less memory)
- Second query: counts total matching documents (for pagination metadata)
- Array destructuring: `[docs, total]`

### Response

```js
      res.json({
        success: true,
        count: docs.length,    // how many in THIS page
        total,                 // how many TOTAL matching records
        page,                  // current page number
        pages: Math.ceil(total / limit),  // total number of pages
        data: docs,            // the actual records
      });
```
- `Math.ceil(total / limit)` — rounds up: 45 records / 20 per page = 3 pages (not 2.25)
- Client knows: "page 2 of 3, showing 20 of 45 results"

---

## GET ONE — `GET /:id`

```js
  router.get('/:id', async (req, res, next) => {
    try {
      const doc = await Model.findOne({
        _id: req.params.id,
        userId: req.user._id,
      }).lean();
```
- `req.params.id` — the `:id` from the URL (e.g., `/api/v1/sleep/507f1f77bcf86cd...`)
- **Two conditions**: must match BOTH the record ID AND the user's ID
- Even if an attacker knows another user's record ID, this query returns nothing (because userId won't match)

```js
      if (!doc) {
        return res.status(404).json({ success: false, error: 'Resource not found' });
      }
      res.json({ success: true, data: doc });
```
- Returns 404 if not found (or if it belongs to another user — same response for both, which is intentional)

---

## CREATE — `POST /`

```js
  router.post('/', async (req, res, next) => {
    try {
      req.body.userId = req.user._id;
```
- **Forces** the userId to be the authenticated user's ID
- Even if someone sends `{ userId: "someone_elses_id" }` in the body, it gets overwritten
- This is the ownership assignment

```js
      if (opts.beforeCreate) {
        await opts.beforeCreate(req);
      }
```
- Optional hook: allows specific routes to modify the request before creating
- Example use: auto-calculate a field, validate a foreign key, etc.

```js
      const doc = await Model.create(req.body);
```
- `Model.create()` — creates and saves a new document
- This triggers all Mongoose pre-save hooks (computed fields get calculated)
- If validation fails → Mongoose throws `ValidationError` → caught by `catch` → `next(err)` → `errorHandler`

```js
      if (opts.afterCreate) {
        await opts.afterCreate(doc, req);
      }
```
- Optional hook: runs side-effects after creation
- Example: update a parent document's aggregate count

```js
      res.status(201).json({ success: true, data: doc });
```
- `201 Created` — proper HTTP status for resource creation

---

## UPDATE — `PUT /:id`

```js
  router.put('/:id', async (req, res, next) => {
    try {
      delete req.body.userId;
      delete req.body._id;
```
- **Security**: prevents users from:
  - Changing `userId` to claim someone else's data
  - Changing `_id` (MongoDB primary key should never change)

```js
      let doc = await Model.findOne({
        _id: req.params.id,
        userId: req.user._id,
      });
```
- Finds the existing document (with ownership check)

```js
      if (!doc) {
        return res.status(404).json({ success: false, error: 'Resource not found' });
      }
```
- Can't update what doesn't exist (or what you don't own)

```js
      Object.assign(doc, req.body);
      doc = await doc.save();
```
- `Object.assign(doc, req.body)` — merges the request body into the existing document
  - New fields get added, existing fields get overwritten
  - Fields NOT in the request body stay unchanged
- `doc.save()` — saves and **triggers pre-save hooks** 
  - This is why we use `findOne` + `save()` instead of `findOneAndUpdate()` 
  - `findOneAndUpdate` would bypass pre-save hooks (no computed fields!)

```js
      res.json({ success: true, data: doc });
```

---

## DELETE — `DELETE /:id`

```js
  router.delete('/:id', async (req, res, next) => {
    try {
      const doc = await Model.findOneAndDelete({
        _id: req.params.id,
        userId: req.user._id,
      });
```
- `findOneAndDelete` — finds AND deletes in one atomic operation
- Both conditions must match (ID + ownership)
- If no match → returns `null`

```js
      if (!doc) {
        return res.status(404).json({ success: false, error: 'Resource not found' });
      }
      res.json({ success: true, data: {} });
```
- Returns empty data on successful deletion

---

```js
  return router;
}
module.exports = crudRouter;
```
- Returns the configured router
- Each module route file calls this with its specific Model:
  ```js
  const crudRouter = require('../utils/crudRouter');
  const Sleep = require('../models/Sleep');
  module.exports = crudRouter(Sleep, { dateField: 'date' });
  ```
