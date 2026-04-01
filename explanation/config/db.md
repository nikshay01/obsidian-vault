# config/db.js — MongoDB Connection

This file exports a single function that connects Mongoose to your MongoDB instance.

---

```js
const mongoose = require('mongoose');
```
- Imports Mongoose — the MongoDB ODM (Object Document Mapper)
- Mongoose lets us define schemas, run queries, and interact with MongoDB using JavaScript objects

---

```js
const connectDB = async () => {
```
- Declares an async function (because `mongoose.connect` returns a Promise)
- `async` lets us use `await` inside

```js
  try {
    const conn = await mongoose.connect(process.env.MONGO_URI);
```
- `process.env.MONGO_URI` — reads the MongoDB connection string from `.env`
- Example: `mongodb://localhost:27017/self-management`
- `await` pauses execution until the connection is established or fails
- `conn` holds the connection object (we use it to log the host)

```js
    console.log(`✅ MongoDB connected: ${conn.connection.host}`);
```
- Logs which MongoDB host we connected to (e.g., `localhost`, `cluster0.mongodb.net`)
- Confirms the connection was successful

```js
  } catch (err) {
    console.error(`❌ MongoDB connection error: ${err.message}`);
    process.exit(1);
```
- If connection fails (wrong URI, MongoDB not running, network error):
  - Logs the error message
  - `process.exit(1)` — **kills the entire Node.js process** with exit code 1
  - Exit code 1 = failure (0 = success)
  - We crash intentionally because a server without a database is useless

```js
  }
};
```

```js
module.exports = connectDB;
```
- Exports the function so `server.js` can call it: `await connectDB()`
