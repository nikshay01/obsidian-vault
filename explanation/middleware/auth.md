# middleware/auth.js — JWT Authentication Guard

This middleware protects routes by verifying JWT tokens. It runs BEFORE every protected route handler.

---

```js
const jwt = require('jsonwebtoken');
const User = require('../models/User');
```
- `jsonwebtoken` — library to create and verify JWT tokens
- `User` — the User model, needed to look up the user from the token's ID

---

```js
const protect = async (req, res, next) => {
```
- Express middleware signature: receives `req` (request), `res` (response), `next` (call to proceed)
- `async` because we do a database lookup inside

```js
  let token;
```
- Declares `token` with `let` (will be assigned conditionally)

---

## Step 1: Extract the Token

```js
  if (
    req.headers.authorization &&
    req.headers.authorization.startsWith('Bearer')
  ) {
    token = req.headers.authorization.split(' ')[1];
  }
```
- Looks for the `Authorization` header in the request
- Expected format: `Authorization: Bearer eyJhbGciOiJIUzI1NiIs...`
- `.startsWith('Bearer')` — confirms the header uses Bearer scheme
- `.split(' ')[1]` — splits `"Bearer <token>"` on space, takes the second part (the actual token)
- If header is missing or doesn't start with "Bearer", `token` stays `undefined`

---

## Step 2: Reject if No Token

```js
  if (!token) {
    return res.status(401).json({
      success: false,
      error: 'Not authorized — no token provided',
    });
  }
```
- If no token was found → immediately return 401 (Unauthorized)
- `return` stops the middleware here — `next()` is never called, so the route handler never runs
- The request dies here for unauthenticated users

---

## Step 3: Verify the Token

```js
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
```
- `jwt.verify()` does two things:
  1. Checks if the token's signature matches using `JWT_SECRET` (proves it wasn't tampered with)
  2. Checks if the token has expired (`exp` claim)
- If either check fails → throws an error → caught by `catch`
- If valid → returns the payload object: `{ id: "507f1f77bcf86cd799439011", iat: ..., exp: ... }`

```js
    req.user = await User.findById(decoded.id).select('-passwordHash');
```
- `decoded.id` — the user's MongoDB `_id` stored in the token during login/register
- `User.findById()` — fetches the full user document from the database
- `.select('-passwordHash')` — EXCLUDES the password hash from the result (the minus `-` means exclude)
- Attaches the user object to `req.user` — now every subsequent middleware/route can access `req.user`

---

## Step 4: Handle Edge Cases

```js
    if (!req.user) {
      return res.status(401).json({
        success: false,
        error: 'Not authorized — user no longer exists',
      });
    }
```
- Edge case: the token is valid but the user was deleted from the database
- This can happen if someone deletes their account but still has an old token

```js
    next();
```
- Everything is valid → call `next()` to proceed to the route handler
- At this point, `req.user` is populated and the route can trust it

---

## Error Handling

```js
  } catch (err) {
    return res.status(401).json({
      success: false,
      error: 'Not authorized — token invalid or expired',
    });
  }
```
- Catches errors from `jwt.verify()`:
  - `JsonWebTokenError` — token was tampered with or malformed
  - `TokenExpiredError` — token's `exp` date has passed
- Returns generic 401 — doesn't reveal WHICH error (security best practice)

---

```js
module.exports = { protect };
```
- Exports as a named export: `const { protect } = require('./middleware/auth')`
- Wrapped in an object so we can add more auth middleware later (e.g., `adminOnly`)
