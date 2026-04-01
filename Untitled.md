Sure! Let me walk you through the entire backend codebase systematically, file by file.

---

## 📁 Project Structure Overview

```
server/
├── .env                  ← Secret config (never commit)
├── .env.example          ← Template for .env
├── package.json          ← Dependencies
├── server.js             ← Entry point (starts everything)
├── config/db.js          ← MongoDB connection
├── middleware/           ← Request interceptors
├── models/               ← Database schemas
├── routes/               ← API endpoints
└── utils/                ← Shared helpers
```

---

## 1. [server.js](cci:7://file:///d:/Nikshay/coding/big%20project/self-management/server/server.js:0:0-0:0) — The Entry Point

This is the file Node.js runs first. It wires **everything** together.

```js
require('dotenv').config();
```
> Loads your `.env` file into `process.env`. Must be the **very first line** so all other files can access env vars.

```js
const express = require('express');
const helmet  = require('helmet');
const cors    = require('cors');
const morgan  = require('morgan');
```
> Imports: Express (web framework), Helmet (security headers), CORS (cross-origin access), Morgan (HTTP request logger).

```js
const app = express();
```
> Creates the Express application — think of it as the HTTP server object.

```js
app.use(helmet());
```
> Adds **10+ security HTTP headers** automatically:
> - `X-Content-Type-Options: nosniff` → browser won't guess MIME types
> - `X-Frame-Options: SAMEORIGIN` → prevents clickjacking
> - `Content-Security-Policy` → controls what scripts can load
> - etc.

```js
app.use(cors());
```
> Allows your frontend (running on a different port/domain) to call this API. Without this, browsers block cross-origin requests.

```js
app.use(globalLimiter);
```
> Applies the **100 requests per 15 minutes** cap to every route globally. Prevents bot abuse.

```js
app.use(express.json({ limit: '10kb' }));
```
> Parses incoming JSON request bodies. The `10kb` limit prevents **large payload attacks** (e.g. someone sending a 100MB JSON to crash your server).

```js
if (process.env.NODE_ENV === 'development') {
  app.use(morgan('dev'));
}
```
> Only logs HTTP requests in development mode. Looks like: `POST /api/v1/auth/login 200 45ms`. Not needed in production.

```js
app.get('/api/v1/health', (_req, res) => {
  res.json({ success: true, status: 'ok', ... });
});
```
> A simple ping endpoint. Useful for uptime monitors — if this returns 200, your server is alive.

```js
app.use('/api/v1/auth/register', authLimiter);
app.use('/api/v1/auth/login', authLimiter);
app.use('/api/v1/auth', authRoutes(protect));
```
> Auth routes get a **stricter rate limiter** (10/15min) to prevent brute-force password attacks. The `authRoutes(protect)` passes the auth middleware into the routes factory.

```js
app.use('/api/v1/sleep', protect, require('./routes/sleep'));
// ... (repeated for all 18 modules)
```
> Every data route first goes through `protect` (JWT check), then the route handler. If no valid token → 401 Unauthorized before reaching the route at all.

```js
app.use((_req, res) => {
  res.status(404).json({ success: false, error: 'Route not found' });
});
app.use(errorHandler);
```
> **404 handler**: catches any request that didn't match a route. **Error handler**: catches any `next(err)` calls from routes.

```js
const start = async () => {
  await connectDB();           // Connect to MongoDB first
  app.listen(PORT, () => { ... }); // THEN start listening
};
start();
```
> Important: we wait for MongoDB to connect **before** accepting requests. If DB fails, the process exits (see `db.js`).

---

## 2. `config/db.js` — Database Connection

```js
const connectDB = async () => {
  try {
    const conn = await mongoose.connect(process.env.MONGO_URI);
    console.log(`✅ MongoDB connected: ${conn.connection.host}`);
  } catch (err) {
    console.error(`❌ MongoDB connection error: ${err.message}`);
    process.exit(1);  // ← KILLS the process if DB fails
  }
};
```
> If MongoDB can't connect (wrong URI, server down), we don't want a broken server running — `process.exit(1)` shuts it down immediately with a non-zero exit code (signals failure to the OS).

---

## 3. `middleware/auth.js` — JWT Guard

This runs **before every protected route**.

```js
const protect = async (req, res, next) => {
  let token;

  if (req.headers.authorization?.startsWith('Bearer')) {
    token = req.headers.authorization.split(' ')[1];
  }
```
> Looks for `Authorization: Bearer eyJhbGciOi...` in the request header. Splits on space and takes the second part (the actual token).

```js
  if (!token) {
    return res.status(401).json({ error: 'Not authorized — no token provided' });
  }
```
> If no token found, immediately reject. Return stops execution here.

```js
  const decoded = jwt.verify(token, process.env.JWT_SECRET);
```
> Verifies the token's signature using your secret key. If the token was tampered with or expired → throws an error → caught by the `catch` block → 401 response.

```js
  req.user = await User.findById(decoded.id).select('-passwordHash');
```
> Looks up the user in the database using the ID stored inside the token. Attaches the user object to `req` so route handlers can access `req.user`. `.select('-passwordHash')` means "exclude the password hash from the result".

```js
  if (!req.user) {
    return res.status(401).json({ error: 'User no longer exists' });
  }
  next();
```
> Edge case: token is valid but the user was deleted from the DB. If user found → `next()` passes control to the route handler.

---

## 4. `middleware/rateLimiter.js` — Rate Limiting

```js
const globalLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,   // 15 minutes in milliseconds
  max: 100,                    // max 100 requests per window
  message: { error: 'Too many requests...' },
  standardHeaders: true,       // Returns RateLimit-* headers
  legacyHeaders: false,        // Disables X-RateLimit-* headers
});
```
> After 100 requests in 15 min from the same IP, further requests get `429 Too Many Requests`. Client gets `RateLimit-Remaining: 0` header so they know.

```js
const authLimiter = rateLimit({ windowMs: 15 * 60 * 1000, max: 10 });
```
> Auth endpoints only get **10 attempts per 15 min**. If someone tries to brute-force your password, they get locked out after 10 tries.

---

## 5. `middleware/errorHandler.js` — Global Error Handler

Express passes any `next(err)` call here. It normalizes different error types:

```js
if (err.name === 'CastError') {
  error = { message: `Resource not found with id: ${err.value}`, statusCode: 404 };
}
```
> Happens when you pass an invalid MongoDB ObjectId (e.g. `/api/v1/sleep/not-a-valid-id`). Mongoose throws `CastError` → we return a clean 404 instead of a 500.

```js
if (err.code === 11000) {
  const field = Object.keys(err.keyValue).join(', ');
  error = { message: `Duplicate field value for: ${field}`, statusCode: 400 };
}
```
> MongoDB's code for **unique constraint violation** (e.g., registering with an existing email). Instead of exposing the raw Mongo error, we send a friendly message.

```js
if (err.name === 'ValidationError') {
  const messages = Object.values(err.errors).map(e => e.message);
  error = { message: messages.join('. '), statusCode: 400 };
}
```
> Mongoose validation failures (e.g. `focusLevel: 15` when max is 10). Collects all field-level messages into one readable string.

```js
...(process.env.NODE_ENV === 'development' && { stack: err.stack })
```
> Only includes the stack trace in development mode — never expose internal traces to end users in production.

---

## 6. `middleware/validate.js` — Input Validation

```js
const validate = (req, res, next) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({
      success: false,
      errors: errors.array().map(e => ({ field: e.path, message: e.msg })),
    });
  }
  next();
};
```
> Used after `express-validator` checks in route definitions. `validationResult(req)` collects all validation failures. Returns them as: `{ errors: [{ field: "email", message: "invalid email" }] }`. If all valid → `next()`.

---

## 7. [utils/computedFields.js](cci:7://file:///d:/Nikshay/coding/big%20project/self-management/server/utils/computedFields.js:0:0-0:0) — Math Helpers

```js
function durationMinutes(start, end) {
  const ms = new Date(end) - new Date(start);
  return ms > 0 ? Math.round(ms / 60000) : 0;
}
```
> Converts two Date objects to a duration in minutes. `new Date(end) - new Date(start)` gives milliseconds. Divide by 60,000 to get minutes. `Math.round` handles partial minutes.

```js
function durationHours(start, end) {
  const ms = new Date(end) - new Date(start);
  return ms > 0 ? parseFloat((ms / 3600000).toFixed(2)) : 0;
}
```
> Same but in hours. `3600000` ms = 1 hour. `toFixed(2)` keeps 2 decimal places, `parseFloat` removes trailing zeros.

```js
function timeDeltaMinutes(actualDate, targetTimeStr) {
  const [h, m] = targetTimeStr.split(':').map(Number);
  const actualMin = actual.getHours() * 60 + actual.getMinutes();
  const targetMin = h * 60 + m;
  return actualMin - targetMin;
}
```
> Used for sleep delta. If target wake is `"06:00"` and you woke at `07:30`, returns `+90` (90 min late). Negative = early.

```js
function autopilotPercent(intentional) {
  return Math.max(0, Math.min(100, 100 - intentional));
}
```
> If intentional use is 70%, autopilot is 30%. `Math.max/min` clamps the result between 0-100.

```js
function formatHoursToHrMin(hours) {
  const h = Math.floor(Math.abs(hours));
  const m = Math.round((Math.abs(hours) - h) * 60);
  const sign = hours < 0 ? '-' : '';
  return `${sign}${h}hr ${m}min`;
}
```
> Converts `1.5` → `"1hr 30min"`, `-0.5` → `"-0hr 30min"`. Used for displaying sleep delta in the UI.

---

## 8. [utils/crudRouter.js](cci:7://file:///d:/Nikshay/coding/big%20project/self-management/server/utils/crudRouter.js:0:0-0:0) — The CRUD Factory

This is the **most important pattern** in the backend. Instead of writing the same GET/POST/PUT/DELETE logic 18 times, this factory generates it for any model.

```js
function crudRouter(Model, opts = {}) {
  const router = express.Router();
  const dateField = opts.dateField || 'date';
```
> Takes a Mongoose model and options. `dateField` is the field used in date queries — different per module (`'date'`, `'timestampStart'`, etc.).

### GET ALL (with filtering, pagination, sorting)
```js
router.get('/', async (req, res, next) => {
  const filter = { userId: req.user._id };
```
> **Always** includes the current user's ID in the query — users can ONLY see their own data.

```js
  if (req.query.date) {
    const d = new Date(req.query.date);
    const next_d = new Date(d);
    next_d.setDate(next_d.getDate() + 1);
    filter[dateField] = { $gte: d, $lt: next_d };
  }
```
> `?date=2026-03-24` translates to a MongoDB range query: `dateField >= 2026-03-24T00:00:00Z AND dateField < 2026-03-25T00:00:00Z`. This handles the timezone-safe "entire day" query.

```js
  const page  = Math.max(1, parseInt(req.query.page) || 1);
  const limit = Math.min(100, Math.max(1, parseInt(req.query.limit) || 20));
  const skip  = (page - 1) * limit;
```
> Pagination math. Default: page 1, 20 per page. Capped at 100 to prevent `?limit=1000000` abuse.

```js
  const [docs, total] = await Promise.all([
    Model.find(filter).sort(sort).skip(skip).limit(limit).lean(),
    Model.countDocuments(filter),
  ]);
```
> Runs both queries **in parallel** (`Promise.all`). `.lean()` returns plain JS objects (faster than Mongoose documents). Returns both the data and the total count for pagination.

### GET ONE (ownership check)
```js
const doc = await Model.findOne({ _id: req.params.id, userId: req.user._id });
```
> The `userId: req.user._id` is the critical part. Even if you know someone else's record ID, you can't access it — the query will return nothing.

### CREATE
```js
req.body.userId = req.user._id;
const doc = await Model.create(req.body);
```
> Injects the current user's ID before saving. User can't fake ownership.

### UPDATE (via `.save()`)
```js
delete req.body.userId;
delete req.body._id;
// ...
Object.assign(doc, req.body);
doc = await doc.save();
```
> `delete req.body.userId` — users can't reassign records to other users. `Object.assign` merges new fields into the existing document. Using `.save()` instead of `findByIdAndUpdate` **triggers pre-save hooks** (computed fields).

---

## 9. [routes/auth.js](cci:7://file:///d:/Nikshay/coding/big%20project/self-management/server/routes/auth.js:0:0-0:0) — Authentication Routes

This is a **factory function** (not just a router):
```js
module.exports = function(protect) {
  const router = express.Router();
  // ...
  return router;
};
```
> Takes the `protect` middleware as a parameter so it can apply it to specific sub-routes internally.

### Register flow:
```js
body('password')
  .isLength({ min: 8 })
  .matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/)
```
> Password must be 8+ chars with at least one lowercase, uppercase, and digit. The regex lookaheads ([(?=...)](cci:1://file:///d:/Nikshay/coding/big%20project/self-management/server/server.js:72:0-78:2)) check without consuming characters.

```js
const existing = await User.findOne({ $or: [{ email }, { username }] });
```
> Checks both email AND username uniqueness in one DB query using MongoDB's `$or` operator.

```js
passwordHash: password  // stored in the model field
```
> The field is named `passwordHash` but we pass the plain password — the `pre('save')` hook in [User.js](cci:7://file:///d:/Nikshay/coding/big%20project/self-management/server/models/User.js:0:0-0:0) hashes it **before** saving. Clean separation of concerns.

### Login flow:
```js
return res.status(401).json({ error: 'Invalid credentials' });
// (same for wrong email AND wrong password)
```
> **Deliberately** returns the same error message for both "email not found" and "wrong password". This prevents **user enumeration** — an attacker can't tell if the email exists.

---

## 10. [models/User.js](cci:7://file:///d:/Nikshay/coding/big%20project/self-management/server/models/User.js:0:0-0:0) — The User Schema

```js
passwordHash: {
  type: String,
  required: true,
  select: false,  // ← KEY
}
```
> `select: false` means `User.findOne(...)` will **never** return `passwordHash` unless you explicitly request it with `.select('+passwordHash')`. The hash will never accidentally leak into API responses.

```js
userSchema.pre('save', async function(next) {
  if (!this.isModified('passwordHash')) return next();
  const salt = await bcrypt.genSalt(12);
  this.passwordHash = await bcrypt.hash(this.passwordHash, salt);
  next();
});
```
> `pre('save')` runs **before** every `.save()`. `isModified` check prevents re-hashing on every unrelated update (e.g. changing your name would re-hash the already-hashed password without this check, corrupting it). Salt rounds of 12 means 2^12 = 4096 bcrypt iterations — slow for attackers, fine for users.

```js
userSchema.methods.matchPassword = async function(password) {
  return await bcrypt.compare(password, this.passwordHash);
};
```
> `bcrypt.compare` is the only safe way to check passwords — you can't "decrypt" a bcrypt hash, you hash the candidate and compare hashes.

```js
userSchema.methods.getSignedJwtToken = function() {
  return jwt.sign({ id: this._id }, process.env.JWT_SECRET, { expiresIn: '30d' });
};
```
> Creates a JWT containing only the user's `_id`. On each request, [auth.js](cci:7://file:///d:/Nikshay/coding/big%20project/self-management/server/routes/auth.js:0:0-0:0) decodes this to know who's making the request. `expiresIn: '30d'` means tokens auto-expire after 30 days.

---

## 11. Example Model: [models/Sleep.js](cci:7://file:///d:/Nikshay/coding/big%20project/self-management/server/models/Sleep.js:0:0-0:0)

```js
sleepSchema.pre('save', function(next) {
  // Auto-compute interruption durations
  this.interruptions.forEach(i => {
    if (i.wakeTime && i.backToSleepTime && !i.durationMinutes) {
      i.durationMinutes = durationMinutes(i.wakeTime, i.backToSleepTime);
    }
  });
  this.totalInterruptionMinutes = this.interruptions.reduce(
    (sum, i) => sum + (i.durationMinutes || 0), 0
  );
```
> The `!i.durationMinutes` check means: only compute if not already manually set (respects manual overrides). The `reduce` sums all interruption durations.

```js
  const rawHours = durationHours(this.sleepStart, this.sleepEnd);
  this.totalSleepHours = parseFloat(
    (rawHours - (this.totalInterruptionMinutes || 0) / 60).toFixed(2)
  );
```
> Total sleep = gross duration minus interruptions. Converts interruption minutes to hours for subtraction.

---

## 12. Example Model: [models/Nutrition.js](cci:7://file:///d:/Nikshay/coding/big%20project/self-management/server/models/Nutrition.js:0:0-0:0)

```js
nutritionSchema.pre('save', function(next) {
  const keys = ['caloriesKcal', 'proteinG', 'carbsG', 'fatG', 'fiberG'];

  this.meals.forEach(meal => {
    meal.mealTotals = {};
    keys.forEach(key => {
      meal.mealTotals[key] = meal.items.reduce(
        (sum, item) => sum + ((item.nutrition?.[key]) || 0), 0
      );
    });
  });

  this.dailyTotals = {};
  keys.forEach(key => {
    this.dailyTotals[key] = this.meals.reduce(
      (sum, meal) => sum + ((meal.mealTotals?.[key]) || 0), 0
    );
  });
});
```
> Two-level aggregation:
> 1. For each meal → sum nutrients across all food items → `mealTotals`
> 2. For the day → sum `mealTotals` across all meals → `dailyTotals`
> The `?.` optional chaining handles items without nutrition data gracefully.

---

## Summary — How a Request Flows

```
Client Request
     ↓
[helmet] — Adds security headers
     ↓
[cors] — Sets CORS headers
     ↓
[globalLimiter] — Checks rate limit
     ↓
[express.json] — Parses body
     ↓
[protect] — Verifies JWT, loads req.user
     ↓
[route handler] — Business logic
     ↓
[Mongoose pre('save')] — Computes fields
     ↓
[MongoDB] — Stores data
     ↓
[errorHandler] — Catches any errors
     ↓
JSON Response
```

Every step is a **middleware** — a function that gets [(req, res, next)](cci:1://file:///d:/Nikshay/coding/big%20project/self-management/server/server.js:72:0-78:2). Calling `next()` passes to the next step, calling `next(err)` skips to the error handler.

---

That's the full codebase! Let me know if you want me to dive deeper into any specific file, model, or concept.