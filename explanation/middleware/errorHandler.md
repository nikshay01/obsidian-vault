# middleware/errorHandler.js — Centralized Error Handler

This middleware catches ALL errors from route handlers. When any route calls `next(err)`, Express skips straight to this function. Registered as the last middleware in [[server|server.js]].

---

```js
const errorHandler = (err, req, res, _next) => {
```
- Express identifies error handlers by having **4 parameters** (err, req, res, next)
- `_next` — prefixed with underscore because we don't call it (we always send a response here)

```js
  let error = { ...err };
  error.message = err.message;
```
- `{ ...err }` — spread operator creates a shallow copy (so we don't mutate the original error)
- `err.message` doesn't get copied by spread (it's on the prototype), so we copy it manually

---

## Dev Logging

```js
  if (process.env.NODE_ENV === 'development') {
    console.error('❌ Error:', err);
  }
```
- Only logs full error details in development mode
- In production, errors go to the response but NOT to the console (or you'd use a proper logging service)

---

## Mongoose CastError (Bad ObjectId)

```js
  if (err.name === 'CastError') {
    error = {
      message: `Resource not found with id: ${err.value}`,
      statusCode: 404,
    };
  }
```
- **When**: you pass an invalid MongoDB ObjectId (e.g., `GET /api/v1/sleep/abc123`)
- Valid ObjectIds are 24 hex characters like `507f1f77bcf86cd799439011`
- Without this handler, the user would see an ugly Mongoose internal error
- We convert it to a clean 404 "not found"

---

## MongoDB Duplicate Key Error

```js
  if (err.code === 11000) {
    const field = Object.keys(err.keyValue).join(', ');
    error = {
      message: `Duplicate field value for: ${field}. Please use another value.`,
      statusCode: 400,
    };
  }
```
- **When**: a unique field constraint is violated (e.g., registering with an email that already exists)
- MongoDB error code `11000` = duplicate key
- `err.keyValue` is an object like `{ email: "test@test.com" }` — we extract the field name
- Returns a user-friendly message instead of the raw MongoDB error

---

## Mongoose Validation Error

```js
  if (err.name === 'ValidationError') {
    const messages = Object.values(err.errors).map((e) => e.message);
    error = {
      message: messages.join('. '),
      statusCode: 400,
    };
  }
```
- **When**: Mongoose schema validation fails (e.g., `focusLevel: 15` when max is 10)
- `err.errors` is an object with one entry per failed field
- `Object.values()` gets all the error objects, `.map(e => e.message)` extracts their messages
- `.join('. ')` combines them: `"focusLevel must be less than 10. taskName is required"`

---

## JWT Errors

```js
  if (err.name === 'JsonWebTokenError') {
    error = { message: 'Invalid token', statusCode: 401 };
  }
  if (err.name === 'TokenExpiredError') {
    error = { message: 'Token expired', statusCode: 401 };
  }
```
- These are usually caught in [[middleware/auth|auth.js]], but if they leak through, we handle them here too
- Both return 401 (Unauthorized)

---

## Final Response

```js
  res.status(error.statusCode || 500).json({
    success: false,
    error: error.message || 'Internal server error',
    ...(process.env.NODE_ENV === 'development' && { stack: err.stack }),
  });
```
- Uses the statusCode we set above, or defaults to 500 (Internal Server Error)
- Uses the message we set, or a generic fallback
- `...(condition && { stack: err.stack })` — conditional spread:
  - In development: includes the full stack trace (helpful for debugging)
  - In production: omits it (stack traces reveal internal code structure — security risk)

---

```js
module.exports = errorHandler;
```
- Exported and used in [[server|server.js]] as the very last `app.use()`
