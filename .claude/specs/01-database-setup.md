# Spec: Step 1 — Database Setup

## 1. Overview

This step replaces the stub in `database/db.py` with a working SQLite implementation. It creates the database connection helper, the schema, and a demo seed so the app has real data to work with.

It is the foundational step of Spendly. Authentication (registration and login), the user profile, and all expense tracking features read from and write to the tables defined here. None of them can be built until this step is complete.

## 2. Depends on

None. This is the first step and has no prerequisites.

## 3. Routes

- No new routes.
- Existing placeholder routes in `app.py` remain unchanged.

## 4. Database Schema

### Table A: `users`

| Column | SQLite Type | Constraints |
|---|---|---|
| id | INTEGER | PRIMARY KEY, AUTOINCREMENT |
| name | TEXT | NOT NULL |
| email | TEXT | NOT NULL, UNIQUE |
| password_hash | TEXT | NOT NULL |
| created_at | TEXT | NOT NULL, DEFAULT current timestamp |

### Table B: `expenses`

| Column | SQLite Type | Constraints |
|---|---|---|
| id | INTEGER | PRIMARY KEY, AUTOINCREMENT |
| user_id | INTEGER | NOT NULL, FOREIGN KEY → users(id) |
| amount | REAL | NOT NULL |
| category | TEXT | NOT NULL (must be one of the fixed categories in Section 10) |
| date | TEXT | NOT NULL, format YYYY-MM-DD required |
| description | TEXT | Nullable |
| created_at | TEXT | NOT NULL, DEFAULT current timestamp |

## 5. Functions to Implement (database/db.py)

| Function | Responsibility |
|---|---|
| `get_db()` | Opens `spendly.db` in the project root; sets `row_factory = sqlite3.Row`; enables `PRAGMA foreign_keys = ON`; returns the connection |
| `init_db()` | Creates both tables using `CREATE TABLE IF NOT EXISTS`; safe to call multiple times |
| `seed_db()` | Checks whether the `users` table already has rows; if yes, returns early. Otherwise inserts one demo user and 8 sample expenses linked to that user |

### seed_db() data requirements

| Item | Requirement |
|---|---|
| Demo user name | Demo User |
| Demo user email | demo@spendly.com |
| Demo user password | demo123, stored hashed via werkzeug |
| Sample expenses | 8 rows, all linked to the demo user |
| Category coverage | All 7 categories from Section 10 appear at least once |
| Date range | Spread across the current month, each in YYYY-MM-DD format |
| Amounts | The 8 amounts total exactly 346.24; Bills has the highest single-category total (Step 05 tests depend on this) |

## 6. Changes to app.py

- Import `get_db`, `init_db`, `seed_db` from `database.db`.
- Call `init_db()` and `seed_db()` inside `app.app_context()` on startup.
- The database must be ready before any route is served.

## 7. Files to Change

- `database/db.py`
- `app.py`

## 8. Files to Create

None.

## 9. Dependencies

- No new pip packages.
- Use `sqlite3` (stdlib).
- Use `werkzeug.security` (already installed and listed in `requirements.txt`).

## 10. Categories (Fixed List)

These are the only valid values for `expenses.category`:

- Food
- Transport
- Bills
- Health
- Entertainment
- Shopping
- Other

## 11. Rules for Implementation

- No ORM, no SQLAlchemy.
- Parameterized queries only — never string formatting in SQL.
- `PRAGMA foreign_keys = ON` on every connection.
- Store `amount` as REAL, not INTEGER.
- Hash passwords with `generate_password_hash` from `werkzeug.security`.
- `seed_db()` must be idempotent (safe to call multiple times).
- Dates must always be YYYY-MM-DD.

## 12. Expected Behavior

### Function behavior

| Function | When called correctly |
|---|---|
| `get_db()` | Returns a working connection to `spendly.db`; rows are accessible by column name; foreign key enforcement is active |
| `init_db()` | Creates the database file if missing and both tables if missing; repeated calls change nothing and raise no errors |
| `seed_db()` | On an empty `users` table, inserts the demo user and 8 expenses; on a populated table, does nothing |

### Database-level constraints

| Constraint | Enforced behavior |
|---|---|
| `users.email` UNIQUE | A second user with the same email is rejected |
| `users` NOT NULL columns | Rows missing name, email, or password_hash are rejected |
| `expenses.user_id` FOREIGN KEY | An expense referencing a non-existent user is rejected |
| `expenses` NOT NULL columns | Rows missing user_id, amount, category, or date are rejected |
| Primary keys | `id` values auto-increment and are never reused |

## 13. Error Handling Expectations

| Scenario | Expected result |
|---|---|
| Duplicate email insert | UNIQUE constraint failure raised |
| Expense with invalid `user_id` | Foreign key constraint failure raised |
| Invalid queries | Clear errors raised for debugging; not silently swallowed |

## 14. Definition of Done

- [ ] Database file created on app startup
- [ ] Both tables exist with correct schema and constraints
- [ ] Demo user exists with hashed password
- [ ] 8 sample expenses exist across all 7 categories, totalling 346.24, with Bills as the top category
- [ ] No duplicate seed data on repeated runs
- [ ] App starts without errors
- [ ] Foreign key enforcement works
- [ ] All queries use parameterized SQL
