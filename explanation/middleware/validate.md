# middleware/validate.js — Input Validation Result Handler

This middleware checks if `express-validator` found any validation errors and returns them in a clean format. Used in [[routes/auth|auth routes]] and other route handlers.

---

```js
const { validationResult } = require('express-validator');
```
- Imports `validationResult` — a function that collects all validation errors from the request
- The actual validation rules (like `body('email').isEmail()`) are defined in route files
- This middleware runs AFTER those rules

---

```js
const validate = (req, res, next) => {
```
- Standard Express middleware signature

```js
  const errors = validationResult(req);
```
- `validationResult(req)` scans the request for any validation failures
- Returns a Result object with an `.isEmpty()` method and `.array()` method

```js
  if (!errors.isEmpty()) {
```
- If there ARE validation errors:

```js
    return res.status(400).json({
      success: false,
      errors: errors.array().map((e) => ({
        field: e.path,
        message: e.msg,
      })),
    });
```
- Returns **400 Bad Request** with a clean error array
- `errors.array()` returns raw errors like: `[{ path: 'email', msg: 'Please provide a valid email', ... }]`
- `.map()` transforms each error into a simpler format: `{ field: 'email', message: '...' }`
- Example response:
  ```json
  {
    "success": false,
    "errors": [
      { "field": "email", "message": "Please provide a valid email" },
      { "field": "password", "message": "Password must be at least 8 characters" }
    ]
  }
  ```
- `return` stops execution — `next()` is never called, so the route handler never runs

```js
  }
  next();
```
- If no errors → `next()` passes control to the next middleware / route handler

---

```js
module.exports = { validate };
```
- Exported as named export
- Used in route files like:
  ```js
  router.post('/register', [body('email').isEmail(), validate], handler);
  ```
  The `validate` function sits at the END of the validation array, AFTER all the rules
