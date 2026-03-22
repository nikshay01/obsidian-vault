# Schema Design Changelog
> Self-Management App — Database Architecture

---

## [v1.3.0] — Logging Strategy Finalized
**Decision: How tasks are logged**

- `todoTasks` → update in place (no separate log collection)
  - status transitions: `todo → in_progress → completed`
  - procrastination fields updated directly on the document
- `habitDefinitions` → never updated for logging, only for config changes
- `habitLogs` → new document created on every log entry, summed per period
- Streak + procrastination fields on definition updated after each log

---

## [v1.2.0] — Habit Procrastination System Clarified
**Decision: Procrastination is conditional on time block**

- Removed: flat `totalMissedPeriods` / `lastMissedAt` from habit procrastination
- Added: `missedDates: [Date]` array for full miss history (always tracked)
- Added: `longestMissStreak` and `currentMissStreak` calculated from `missedDates`
- Added: `procrastination.enabled` — auto-set to `true` only when `timeBlock.enabled = true`
- `habitLogs.timeBlock.minutesLate` → `null` when no time block is set
- Anytime habits → miss tracking only, no clock
- Time-blocked habits → full procrastination clock + miss tracking

---

## [v1.1.0] — Core Schema Decisions Locked
**All finalized from Q&A session**

### Procrastination (Todo)
- Trigger: both auto (past `estimatedStartTime`) and manual override
- Clock starts exactly when `estimatedStartTime` is hit and task is not started
- `checkpoints[]` array for procrastination shape over time (recorded on app open/close)
- `totalMinutes` accumulates until completion, then frozen permanently

### Habit Logs
- Multiple log entries allowed per period
- Entries are **summed** (`periodTotalValue` = running sum)
- `completionPercent = (periodTotalValue / targetValue) * 100`

### Streak Logic
- Default `allowedMissesBeforeBreak: 1` (1 miss allowed before streak breaks)
- Configurable per habit via `streak.allowedMissesBeforeBreak`

### Categories
- Predefined + user can add custom
- Stored in separate `categories` collection with slugs
- Referenced by slug across all other collections

### Exercise Sessions
- Workout deferred — walk and run only for now
- Metrics: `distanceKm`, `steps`, `avgPaceMinPerKm`, `caloriesBurned`, `mood`, `energy`

---

## [v1.0.0] — Initial Schema Architecture
**Collections defined**

| Collection | Purpose |
|---|---|
| `categories` | Shared lookup for category/subcategory/project/subproject |
| `todoTasks` | One-time tasks with procrastination tracking |
| `habitDefinitions` | Recurring habit/task definitions |
| `habitLogs` | Per-entry logs for habits (child of habitDefinitions) |
| `exerciseSessions` | Walk and run session tracking |
| `dailySummaries` | Aggregated daily snapshot from all modules |

### Predefined Categories
- Personal & Habits
- Creative & Side Projects
- Finance
- Self Development
- Professional Work
- Academic
- Health & Fitness

### Key Design Principles
- `todoTasks` and `habitDefinitions` share the same classification fields:
  `category`, `subcategory`, `project`, `subproject`, `priority`, `labels`
- All collections feed into `dailySummaries` for unified dashboard data
- Habit completion is versatile: `"boolean" | "duration" | "percentage" | "fraction" | "count" | "custom"`
- Procrastination is tracked at two granularities:
  - **Minute-level** for todos (time-sensitive)
  - **Period-level** (day/week) for habits

---

## Pending / Not Yet Finalized

- [ ] Charts and graphs system — which data points feed which graphs
- [ ] `dailySummaries` aggregation logic — when and how it runs
- [ ] Todo `sessions[]` array — multi-sitting support for a single task
- [ ] Workout schema — deferred
- [ ] API route design
- [ ] Frontend UI
