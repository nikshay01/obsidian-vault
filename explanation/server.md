# server.js — Entry Point

This is the main file that starts the entire backend. It wires together all middleware, routes, and the database connection.

---

```js
require('dotenv').config();
```
- Loads all variables from the `.env` file into `process.env`
- Must be the **very first line** so every other file can access `MONGO_URI`, `JWT_SECRET`, etc.

```js
const express = require('express');
const helmet = require('helmet');
const cors = require('cors');
const morgan = require('morgan');
```
- **express** — the web framework, handles HTTP requests/responses
- **helmet** — automatically sets ~10 security HTTP headers (prevents clickjacking, sniffing, etc.)
- **cors** — allows cross-origin requests (so your frontend on a different port can call this API)
- **morgan** — logs every HTTP request to the console (method, URL, status, time)

```js
const connectDB = require('./config/db');
const errorHandler = require('./middleware/errorHandler');
const { protect } = require('./middleware/auth');
const { globalLimiter, authLimiter } = require('./middleware/rateLimiter');
```
- Imports our custom modules:
  - `connectDB` — function to connect to MongoDB
  - `errorHandler` — catches all errors and sends clean JSON responses
  - `protect` — JWT authentication middleware
  - `globalLimiter` / `authLimiter` — rate limiting middleware

```js
const app = express();
```
- Creates the Express application object — this IS the server

---

## Security Middleware

```js
app.use(helmet());
```
- Adds security headers to every response:
  - `X-Content-Type-Options: nosniff` — browser won't guess MIME types
  - `X-Frame-Options: SAMEORIGIN` — prevents clickjacking via iframes
  - `Content-Security-Policy` — controls what resources can load
  - `Strict-Transport-Security` — forces HTTPS
  - and more...

```js
app.use(cors());
```
- Adds `Access-Control-Allow-Origin` headers
- Without this, browsers block requests from `localhost:3000` (frontend) to `localhost:5000` (API)

```js
app.use(globalLimiter);
```
- Applies **100 requests per 15 minutes** limit to ALL routes
- Prevents bot abuse and DDoS attacks
- After exceeding limit → returns `429 Too Many Requests`

---

## Body Parsing

```js
app.use(express.json({ limit: '10kb' }));
```
- Parses JSON request bodies (`Content-Type: application/json`)
- `limit: '10kb'` — rejects payloads larger than 10 kilobytes (prevents large payload attacks)
- After this middleware, `req.body` contains the parsed JSON object

```js
app.use(express.urlencoded({ extended: false }));
```
- Parses URL-encoded form data (`Content-Type: application/x-www-form-urlencoded`)
- `extended: false` — uses the simpler `querystring` library (no nested objects)

---

## Logging

```js
if (process.env.NODE_ENV === 'development') {
  app.use(morgan('dev'));
}
```
- Only logs in development mode (not in production)
- Output looks like: `POST /api/v1/auth/login 200 45.123ms`
- Shows: method, URL, status code, response time

---

## Health Check

```js
app.get('/api/v1/health', (_req, res) => {
  res.json({ success: true, status: 'ok', timestamp: new Date().toISOString() });
});
```
- Simple ping endpoint — if this returns 200, the server is alive
- Used by monitoring services (e.g., uptime robots)
- `_req` — underscore prefix means "this parameter is unused but required by Express"

---

## Auth Routes

```js
app.use('/api/v1/auth/register', authLimiter);
app.use('/api/v1/auth/login', authLimiter);
```
- Applies **stricter rate limiting** (10 requests per 15 min) to register and login endpoints
- Prevents brute-force password attacks

```js
app.use('/api/v1/auth', authRoutes(protect));
```
- Mounts the auth router at `/api/v1/auth`
- Passes `protect` middleware so the router can internally apply it to `/me` and `/password` routes
- Register and login are public; `/me` and `/password` are protected

---

## Protected Data Routes

```js
app.use('/api/v1/sleep',             protect, require('./routes/sleep'));
app.use('/api/v1/work-sessions',     protect, require('./routes/workSessions'));
app.use('/api/v1/meditation',        protect, require('./routes/meditation'));
// ... (same pattern for all 18 modules)
```
- Every data route has `protect` as the first middleware
- `protect` verifies the JWT token → if invalid, returns 401 immediately
- If token valid → `req.user` is set → route handler runs
- Each `require('./routes/...')` loads the CRUD router for that module

---

## Error Handling

```js
app.use((_req, res) => {
  res.status(404).json({ success: false, error: 'Route not found' });
});
```
- **404 catch-all** — if no route matched the request, this runs
- Must be AFTER all route definitions (it catches everything that "falls through")

```js
app.use(errorHandler);
```
- **Global error handler** — catches any `next(err)` calls from route handlers
- Must be LAST middleware (4 arguments: `err, req, res, next`)

---

## Server Startup

```js
const PORT = process.env.PORT || 5000;
```
- Uses PORT from `.env`, defaults to 5000

```js
const start = async () => {
  await connectDB();
  app.listen(PORT, () => {
    console.log(`🚀 Server running in ${process.env.NODE_ENV} mode on port ${PORT}`);
    console.log(`📋 API base: http://localhost:${PORT}/api/v1`);
  });
};
start();
```
- `await connectDB()` — waits for MongoDB connection before doing anything
- If DB fails → `process.exit(1)` in `db.js` kills the server
- `app.listen(PORT)` — starts accepting HTTP connections on that port
- `start()` is called immediately (self-invoking async pattern)
