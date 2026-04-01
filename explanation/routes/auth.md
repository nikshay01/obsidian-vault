# routes/auth.js — Authentication Routes

This file handles user registration, login, profile management, and password changes. It's a **factory function** that accepts the `protect` middleware. Registered in [[server|server.js]].

---

```js
const express = require('express');
const { body } = require('express-validator');
const { validate } = require('../middleware/validate');
const User = require('../models/User');
```
- `express` — web framework
- `body` — from express-validator, creates validation rules for request body fields
- `validate` — our custom middle ware that checks validation results (see [[middleware/validate]])
- `User` — the [[models/User|User]] model for database operations

```js
module.exports = function (protect) {
const router = express.Router();
```
- This is a **factory function** — it receives `protect` (from [[middleware/auth]]) as a parameter and returns a router
- This pattern allows the [[server|server.js]] to inject the protect middleware
- Creating a fresh router instance inside the function

---

## POST /register

```js
router.post(
  '/register',
  [
```
- Route: `POST /api/v1/auth/register`
- The array `[...]` contains validation middleware that runs BEFORE the handler

### Validation Rules

```js
    body('username')
      .trim()
      .isLength({ min: 3, max: 30 })
      .withMessage('Username must be 3-30 characters')
      .matches(/^[a-zA-Z0-9_]+$/)
      .withMessage('Username can only contain letters, numbers and underscores'),
```
- `.trim()` — removes whitespace from start/end
- `.isLength({ min: 3, max: 30 })` — enforces length range
- `.matches(/^[a-zA-Z0-9_]+$/)` — regex: only allows letters, digits, and underscores
  - `^` — start of string, `$` — end of string, `+` — one or more characters
  - `[a-zA-Z0-9_]` — character class: lowercase, uppercase, digits, underscore
- `.withMessage()` — custom error message (used instead of the auto-generated one)

```js
    body('email').isEmail().withMessage('Please provide a valid email').normalizeEmail(),
```
- `.isEmail()` — checks valid email format
- `.normalizeEmail()` — sanitizer: `Test@Gmail.COM` → `test@gmail.com`

```js
    body('password')
      .isLength({ min: 8 })
      .withMessage('Password must be at least 8 characters')
      .matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/)
      .withMessage('Password must contain uppercase, lowercase and number'),
```
- `.isLength({ min: 8 })` — minimum 8 characters
- The regex uses **lookaheads** to check multiple conditions without consuming characters:
  - `(?=.*[a-z])` — at least one lowercase letter exists somewhere
  - `(?=.*[A-Z])` — at least one uppercase letter exists somewhere
  - `(?=.*\d)` — at least one digit exists somewhere
  - `^` — anchored to start (all lookaheads check from the beginning)

```js
    validate,
  ],
```
- `validate` middleware runs LAST in the array — checks if any of the above rules failed

### Handler

```js
  async (req, res, next) => {
    try {
      const { username, email, password, name } = req.body;
```
- Destructures the validated fields from the request body

```js
      const existing = await User.findOne({ $or: [{ email }, { username }] });
```
- Checks if email OR username already exists in the database
- `$or` — MongoDB operator: matches if ANY condition is true
- Single query instead of two separate queries (more efficient)

```js
      if (existing) {
        return res.status(400).json({
          success: false,
          error: existing.email === email ? 'Email already registered' : 'Username already taken',
        });
      }
```
- If found, returns which one was the duplicate
- We CAN be specific here (unlike login) because this is registration — the user knows their own email

```js
      const user = await User.create({
        username,
        email,
        passwordHash: password,
        name: name || username,
      });
```
- `passwordHash: password` — we pass the PLAIN password; the `pre('save')` hook in User.js hashes it automatically
- `name: name || username` — if no name provided, use the username as default

```js
      sendTokenResponse(user, 201, res);
```
- Sends back a JWT token (see helper function below)
- `201` = Created status code

---

## POST /login

```js
router.post(
  '/login',
  [
    body('email').isEmail().withMessage('Please provide a valid email').normalizeEmail(),
    body('password').notEmpty().withMessage('Password is required'),
    validate,
  ],
```
- Lighter validation than register — just checks email format and password presence

```js
  async (req, res, next) => {
    try {
      const { email, password } = req.body;

      const user = await User.findOne({ email }).select('+passwordHash');
```
- `.select('+passwordHash')` — overrides `select: false` on the schema
- Without this, `user.passwordHash` would be `undefined` and `matchPassword` would always fail
- The `+` prefix means "include this normally-excluded field"

```js
      if (!user) {
        return res.status(401).json({ success: false, error: 'Invalid credentials' });
      }
```
- Returns generic "Invalid credentials" — NOT "email not found"
- **Security**: if we said "email not found", attackers could check which emails are registered

```js
      const isMatch = await user.matchPassword(password);
      if (!isMatch) {
        return res.status(401).json({ success: false, error: 'Invalid credentials' });
      }
```
- `matchPassword` — bcrypt.compare() the plain password with the hash
- Same generic error message for wrong password — attacker can't tell the difference

```js
      sendTokenResponse(user, 200, res);
```
- Login success → return JWT token

---

## GET /me (Protected)

```js
router.get('/me', protect, async (req, res) => {
  res.json({ success: true, data: req.user });
});
```
- `protect` middleware runs first → verifies JWT → sets `req.user`
- Simply returns the user's profile (password hash is already excluded by [[middleware/auth|protect]])

---

## PUT /me (Protected)

```js
router.put('/me', protect, async (req, res, next) => {
  try {
    delete req.body.passwordHash;
    delete req.body.email;
    delete req.body._id;
```
- **Security**: prevents updating sensitive fields through this endpoint
- `passwordHash` — must use the `/password` endpoint (requires current password verification)
- `email` — should require email verification (separate flow, not implemented)
- `_id` — primary key should never change

```js
    const user = await User.findByIdAndUpdate(req.user._id, req.body, {
      new: true,
      runValidators: true,
    });
```
- `findByIdAndUpdate` — finds by the authenticated user's ID and applies changes
- `new: true` — returns the UPDATED document (default returns the old one)
- `runValidators: true` — applies Mongoose schema validation on the update
  - By default, `findByIdAndUpdate` skips validation! This flag is important.

---

## PUT /password (Protected)

```js
router.put(
  '/password',
  protect,
  [
    body('currentPassword').notEmpty().withMessage('Current password is required'),
    body('newPassword')
      .isLength({ min: 8 })
      .matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/)
      ...
    validate,
  ],
```
- `protect` runs first (must be logged in)
- Then validates both `currentPassword` and `newPassword`

```js
    const user = await User.findById(req.user._id).select('+passwordHash');
```
- Fetches the user WITH the password hash (needed for comparison)

```js
    const isMatch = await user.matchPassword(req.body.currentPassword);
    if (!isMatch) {
      return res.status(401).json({ success: false, error: 'Current password is incorrect' });
    }
```
- Verifies the current password before allowing a change
- Even if someone stole the JWT, they still need the old password

```js
    user.passwordHash = req.body.newPassword;
    await user.save();
```
- Sets the new password (plain text) → `pre('save')` hook hashes it
- `isModified('passwordHash')` in the hook will be `true`, so it hashes

```js
    sendTokenResponse(user, 200, res);
```
- Returns a new JWT (good practice: old tokens still work until expiry)

---

## Helper Function

```js
function sendTokenResponse(user, statusCode, res) {
  const token = user.getSignedJwtToken();
```
- Calls the User model's method to generate a JWT containing `{ id: user._id }`

```js
  res.status(statusCode).json({
    success: true,
    token,
    data: {
      id: user._id,
      username: user.username,
      email: user.email,
      name: user.name,
    },
  });
}
```
- Returns the token plus basic user info
- Does NOT return the full user document (minimizes data exposure)
- Client stores this token and sends it in `Authorization: Bearer <token>` on subsequent requests

---

```js
return router;
};
```
- Returns the configured router from the factory function
- [[server|server.js]] calls: `app.use('/api/v1/auth', authRoutes(protect))`
