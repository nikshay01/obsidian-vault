# utils/computedFields.js — Shared Calculation Helpers

These utility functions are used by Mongoose pre-save hooks across multiple models to auto-compute derived fields. Used by [[models/Sleep]], [[models/WorkSession]], [[models/GamingSession]], [[models/HobbySession]], [[models/PainLog]], [[models/Devotion]], [[models/Nutrition]], [[models/ScreenTime]], [[models/HabitLog]], and [[models/Todo]].

---

```js
/**
 * Duration in minutes between two Date objects.
 */
function durationMinutes(start, end) {
```
- Takes two Date objects (or date strings) and returns the duration between them in minutes

```js
  if (!start || !end) return null;
```
- If either date is missing, return null (can't compute without both)

```js
  const ms = new Date(end) - new Date(start);
```
- `new Date(end) - new Date(start)` — JavaScript auto-converts Dates to timestamps (milliseconds since 1970) when you subtract them
- Result is the difference in milliseconds

```js
  return ms > 0 ? Math.round(ms / 60000) : 0;
```
- `ms / 60000` — converts milliseconds to minutes (60,000 ms = 1 minute)
- `Math.round()` — rounds to nearest whole minute
- `ms > 0 ?` — if end is before start (negative duration), return 0 instead of a negative number
- Used by: [[models/Sleep]], [[models/WorkSession]], [[models/GamingSession]], [[models/HobbySession]], [[models/PainLog]], [[models/Todo]]

---

```js
function durationHours(start, end) {
  if (!start || !end) return null;
  const ms = new Date(end) - new Date(start);
  return ms > 0 ? parseFloat((ms / 3600000).toFixed(2)) : 0;
}
```
- Same logic but returns **hours** instead of minutes
- `3600000` ms = 1 hour
- `.toFixed(2)` — keeps 2 decimal places (e.g., `8.25` hours)
- `parseFloat()` — removes trailing zeros (`"8.00"` → `8`, `"8.50"` → `8.5`)
- Used by: [[models/Sleep]] for `totalSleepHours` computation

---

```js
function timeDeltaMinutes(actualDate, targetTimeStr) {
```
- Computes how many minutes early/late something was compared to a target time
- Example: target wake time is `"06:00"`, actual wake was 7:30 AM → returns `+90`

```js
  if (!actualDate || !targetTimeStr) return null;
```
- Guard clause for missing values

```js
  const actual = new Date(actualDate);
```
- Converts to a Date object if it isn't already

```js
  const [h, m] = targetTimeStr.split(':').map(Number);
```
- Splits `"06:00"` into `["06", "00"]`, then `.map(Number)` converts to `[6, 0]`
- Destructures into `h = 6` and `m = 0`

```js
  const actualMin = actual.getHours() * 60 + actual.getMinutes();
```
- Converts actual time to total minutes since midnight
- 7:30 AM → `7 * 60 + 30` = `450` minutes

```js
  const targetMin = h * 60 + m;
```
- Converts target time to total minutes since midnight
- `"06:00"` → `6 * 60 + 0` = `360` minutes

```js
  return actualMin - targetMin;
```
- Returns the difference: `450 - 360 = 90` (90 minutes late)
- Positive = late/over, Negative = early/under
- Used by: [[models/Sleep]] for sleep/wake time delta computation

---

```js
function sumByKey(arr, key) {
  if (!Array.isArray(arr)) return 0;
  return arr.reduce((acc, item) => acc + (Number(item[key]) || 0), 0);
}
```
- Sums a specific property across an array of objects
- Example: `sumByKey([{cal: 200}, {cal: 350}], 'cal')` → `550`
- `Number(item[key]) || 0` — converts to number, defaults to 0 if NaN/null/undefined
- `.reduce()` — accumulates the sum starting from 0

---

```js
function autopilotPercent(intentional) {
  if (intentional == null) return null;
  return Math.max(0, Math.min(100, 100 - intentional));
}
```
- If intentional screen time is 70%, autopilot is 30%
- `100 - intentional` — simple subtraction
- `Math.max(0, Math.min(100, ...))` — clamps result between 0 and 100
  - `Math.min(100, x)` — ensures result is ≤ 100
  - `Math.max(0, x)` — ensures result is ≥ 0
- `intentional == null` — checks for both `null` AND `undefined` (loose equality)
- Used by: [[models/ScreenTime]] pre-save hook

---

```js
function completionPercent(logged, target) {
  if (!target || target === 0) return 0;
  return parseFloat(((logged / target) * 100).toFixed(1));
}
```
- Calculates habit completion percentage
- Example: logged 2 hours out of 3 target → `(2/3) * 100` = `66.7%`
- `!target || target === 0` — prevents division by zero
- `.toFixed(1)` — one decimal place
- `parseFloat()` — removes trailing zero (`"100.0"` → `100`)
- Used by: [[models/HabitLog]] pre-save hook

---

```js
function formatHoursToHrMin(hours) {
  if (hours == null) return '';
  const h = Math.floor(Math.abs(hours));
  const m = Math.round((Math.abs(hours) - h) * 60);
  const sign = hours < 0 ? '-' : '';
  return `${sign}${h}hr ${m}min`;
}
```
- Converts float hours to human-readable format
- `Math.abs(hours)` — works with the absolute value (handles negatives)
- `Math.floor()` — extracts whole hours: `1.5` → `1`
- `(Math.abs(hours) - h) * 60` — extracts remaining minutes: `(1.5 - 1) * 60` = `30`
- `Math.round()` — rounds minutes to nearest whole number
- `sign` — adds `-` prefix for negative values
- Examples: `1.5` → `"1hr 30min"`, `-0.5` → `"-0hr 30min"`
- Used by: [[models/Sleep]] for human-readable sleep hour delta display

---

```js
module.exports = {
  durationMinutes,
  durationHours,
  timeDeltaMinutes,
  sumByKey,
  autopilotPercent,
  completionPercent,
  formatHoursToHrMin,
};
```
- Exports all functions for use in model pre-save hooks
