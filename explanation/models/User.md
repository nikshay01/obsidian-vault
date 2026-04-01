# models/User.js — User Schema & Authentication

The most complex model — handles identity, passwords, JWT tokens, preferences, goals, and aggregates.

---

## Imports

```js
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');
```
- `mongoose` — MongoDB ODM for schema definition
- `bcryptjs` — password hashing library (pure JS version, no native deps)
- `jsonwebtoken` — creates and verifies JWT tokens

---

## Schema Definition

```js
const userSchema = new mongoose.Schema(
  {
```
- Creates a new Mongoose schema — defines the shape of User documents in MongoDB

### Identity Fields

```js
    username: {
      type: String,
      required: [true, 'Username is required'],
      unique: true,
      trim: true,
      minlength: 3,
      maxlength: 30,
      match: [/^[a-zA-Z0-9_]+$/, 'Username can only contain letters, numbers and underscores'],
    },
```
- `type: String` — field type
- `required: [true, 'message']` — validation: field must exist, custom error message
- `unique: true` — creates a MongoDB unique index (no two users can have same username)
- `trim: true` — auto-strips whitespace from start/end before saving
- `minlength: 3, maxlength: 30` — string length validation
- `match: [regex, message]` — must match this regex pattern

```js
    email: {
      type: String,
      required: [true, 'Email is required'],
      unique: true,
      lowercase: true,
      trim: true,
      match: [/^\S+@\S+\.\S+$/, 'Please provide a valid email'],
    },
```
- `lowercase: true` — auto-converts to lowercase before saving (`Test@Gmail.COM` → `test@gmail.com`)
- `match: /^\S+@\S+\.\S+$/` — basic email format: `nonspace@nonspace.nonspace`

```js
    passwordHash: {
      type: String,
      required: [true, 'Password is required'],
      minlength: 8,
      select: false,
    },
```
- **`select: false`** — the most important security setting for this field
  - `User.findOne(...)` will NEVER return `passwordHash` unless explicitly requested with `.select('+passwordHash')`
  - Prevents accidental leaking of password hashes in API responses

### Organization System

```js
    categories: [
      {
        name: { type: String, required: true },
        parentCategoryId: { type: mongoose.Schema.Types.ObjectId, default: null },
      },
    ],
```
- Array of embedded subdocuments
- `parentCategoryId: null` means top-level category
- Non-null value = nested under another category (hierarchical structure)
- `mongoose.Schema.Types.ObjectId` — MongoDB's unique ID type

### Configuration

```js
    meditation: {
      types: {
        type: [String],
        default: ['breath', 'mantra', 'guided', 'silent', 'walking', 'body_scan', 'mindfulness'],
      },
```
- `type: [String]` — array of strings
- `default: [...]` — if not provided, starts with these predefined types
- User can add/remove from this list via profile updates

### Preferences

```js
    preferences: {
      activeModules: {
        mood: { type: Boolean, default: true },
        // ... more modules
      },
      startOfDayOffsetHours: { type: Number, default: 0, min: 0, max: 6 },
    },
```
- `activeModules` — toggles for each module (user can disable modules they don't use)
- `startOfDayOffsetHours` — custom day boundary (e.g., 3 = day starts at 3 AM, for night owls)

### Timestamps

```js
  },
  { timestamps: true }
);
```
- `timestamps: true` — Mongoose auto-adds `createdAt` and `updatedAt` fields
- Updated automatically on every `.save()` call

---

## Pre-Save Hook: Password Hashing

```js
userSchema.pre('save', async function (next) {
```
- `pre('save')` — runs BEFORE every `.save()` call on a User document
- `function` (not arrow) — because we need `this` to refer to the document

```js
  if (!this.isModified('passwordHash')) return next();
```
- **Critical check**: only hash if the password field was actually changed
- Without this, updating a user's name would re-hash the already-hashed password, corrupting it
- `isModified()` — Mongoose method that tracks which fields changed since last save
- If password not modified → skip to `next()` (proceed without hashing)

```js
  const salt = await bcrypt.genSalt(12);
```
- Generates a random salt with cost factor 12
- Cost factor 12 means 2^12 = 4,096 iterations of the hashing function
- Higher = slower = more secure against brute-force
- 12 is a good balance between security and performance (~250ms per hash)

```js
  this.passwordHash = await bcrypt.hash(this.passwordHash, salt);
```
- Hashes the plain password with the salt
- Output looks like: `$2a$12$LJ3m4ys3Rl5QFrNvPm2nTeY7QkPf0uHG9u4K1RWQB...`
  - `$2a$` — bcrypt algorithm identifier
  - `12$` — cost factor
  - Next 22 chars — the salt
  - Remaining — the hash

```js
  next();
```
- Proceeds to save the document to MongoDB

---

## Instance Methods

```js
userSchema.methods.matchPassword = async function (enteredPassword) {
  return await bcrypt.compare(enteredPassword, this.passwordHash);
};
```
- Called on a user instance: `user.matchPassword('myPassword123')`
- `bcrypt.compare()` — hashes the entered password with the same salt from the stored hash, then compares
- Returns `true` if match, `false` otherwise
- You CANNOT decrypt a bcrypt hash — you can only compare

```js
userSchema.methods.getSignedJwtToken = function () {
  return jwt.sign({ id: this._id }, process.env.JWT_SECRET, {
    expiresIn: process.env.JWT_EXPIRE || '30d',
  });
};
```
- Creates a JWT token containing: `{ id: "the_user_mongodb_id" }`
- `process.env.JWT_SECRET` — the secret key used to sign (must be kept secret!)
- `expiresIn: '30d'` — token expires in 30 days
- The resulting token is a string like: `eyJhbGciOiJIUzI1NiIs.eyJpZCI6IjY1ZjFhM.SflKxwRJSMeKKF2Q`
  - Part 1: header (algorithm)
  - Part 2: payload (our `{ id }` + expiry)
  - Part 3: signature (proves it wasn't tampered with)

---

```js
module.exports = mongoose.model('User', userSchema);
```
- Registers the schema as a Mongoose model named `'User'`
- MongoDB will create a collection called `users` (auto-pluralized)
- Everywhere in the code, `require('./models/User')` gives you the Model class
