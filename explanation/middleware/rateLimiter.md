# middleware/rateLimiter.js — Rate Limiting

Prevents abuse by limiting how many requests an IP address can make in a time window.

---

```js
const rateLimit = require('express-rate-limit');
```
- Imports the rate limiting library

---

## Global Rate Limiter

```js
const globalLimiter = rateLimit({
```
- Creates a rate limiter instance with the following config:

```js
  windowMs: 15 * 60 * 1000,
```
- `windowMs` — the time window in milliseconds
- `15 * 60 * 1000` = 15 minutes × 60 seconds × 1000 milliseconds = **900,000ms = 15 minutes**

```js
  max: 100,
```
- Maximum **100 requests** per IP within the 15-minute window
- After 100 requests → all further requests get blocked until the window resets

```js
  message: {
    success: false,
    error: 'Too many requests, please try again after 15 minutes',
  },
```
- The JSON response sent when the limit is exceeded
- Client receives HTTP status **429 (Too Many Requests)**

```js
  standardHeaders: true,
```
- Adds `RateLimit-Limit`, `RateLimit-Remaining`, and `RateLimit-Reset` headers to every response
- Clients can read these to know how many requests they have left

```js
  legacyHeaders: false,
```
- Disables the older `X-RateLimit-*` headers (we use the modern standard ones instead)

```js
});
```

---

## Auth Rate Limiter

```js
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 10,
```
- Same 15-minute window but only **10 requests** allowed
- Much stricter because auth endpoints are the main target for brute-force attacks
- After 10 failed login attempts → attacker is locked out for 15 minutes

```js
  message: {
    success: false,
    error: 'Too many login attempts, please try again after 15 minutes',
  },
  standardHeaders: true,
  legacyHeaders: false,
});
```
- Same structure as global, just with different message and lower limit

---

```js
module.exports = { globalLimiter, authLimiter };
```
- Exports both limiters
- `globalLimiter` is applied to ALL routes in `server.js`
- `authLimiter` is applied only to `/auth/register` and `/auth/login`
