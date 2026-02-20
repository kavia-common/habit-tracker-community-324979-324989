# Habit Buddy DB schema verification (PostgreSQL)

This file documents the schema verification for the `habit_buddy_database` container and its fit with the `habit_buddy_backend` FastAPI endpoints.

## Connection (authoritative)

From `habit_buddy_database/db_connection.txt`:

- `psql postgresql://appuser:dbuser123@localhost:5000/myapp`

Connectivity check executed:

- `SELECT current_database(), current_user, inet_server_port();`
  - `db=myapp`, `usr=appuser`, `port=5000`

## Tables present (public schema)

Verified via:

- `SELECT tablename FROM pg_tables WHERE schemaname='public' ORDER BY tablename;`

Tables found (14):

- `users`
- `habits`
- `habit_checkins`
- `habit_streaks`
- `groups`
- `group_members`
- `challenges`
- `challenge_participants`
- `badges`
- `user_badges`
- `notifications`
- `feed_posts`
- `feed_post_likes`
- `feed_post_comments`

## Constraints present (PK/FK/unique)

Verified via:

- `SELECT conname, conrelid::regclass AS table_name, pg_get_constraintdef(oid) AS def FROM pg_constraint WHERE connamespace='public'::regnamespace ORDER BY conrelid::regclass::text, conname;`

Key items required by backend logic:

- `habit_checkins`: `UNIQUE (habit_id, checkin_date)` ensures one check-in per habit per day (backend returns 409 on duplicates).
- `feed_post_likes`: `PRIMARY KEY (post_id, user_id)` enables idempotent likes (backend treats unique violation as "already liked").

## Indexes present

Verified via:

- `SELECT indexname, tablename FROM pg_indexes WHERE schemaname='public' ORDER BY tablename, indexname;`

Notable indexes (in addition to PK/unique constraints):

- `idx_habits_user_id` on `habits`
- `idx_checkins_habit_date` on `habit_checkins`
- `idx_checkins_user_date` on `habit_checkins`
- `idx_group_members_user` on `group_members`
- `idx_feed_posts_created` on `feed_posts`

## Seed data present

Row counts verified:

- `users`: 3
- `habits`: 3
- `groups`: 1
- `challenges`: 1
- `badges`: 2
- `feed_posts`: 1

## Backend integration checks (02.01)

### DB schema alignment change applied

Mismatch found vs backend ORM intent:

- `users.timezone` had a DB default `'UTC'::text`, but backend ORM treats timezone as nullable/no server default.

Applied (one statement):

- `ALTER TABLE users ALTER COLUMN timezone DROP DEFAULT;`

### Remaining gap: auth endpoints returning 500 appears non-DB

During smoke calls against backend `:3001`, both:

- `POST /auth/register`
- `POST /auth/login`

returned `500 Internal Server Error` with no response body.

Given the DB schema supports inserts (manual insert relying on `gen_random_uuid()` works), this remaining 500 is most likely due to backend runtime configuration (e.g., `JWT_SECRET_KEY` missing, which would raise a RuntimeError when creating tokens) rather than missing DB tables/columns/constraints.

## How to re-run these checks

Run each statement as a single `psql -c "..."` command, for example:

- `psql postgresql://appuser:dbuser123@localhost:5000/myapp -c "SELECT tablename FROM pg_tables WHERE schemaname='public' ORDER BY tablename;"`
- `psql postgresql://appuser:dbuser123@localhost:5000/myapp -c "SELECT indexname, tablename FROM pg_indexes WHERE schemaname='public' ORDER BY tablename, indexname;"`
- `psql postgresql://appuser:dbuser123@localhost:5000/myapp -c "SELECT COUNT(*) FROM users;"`
