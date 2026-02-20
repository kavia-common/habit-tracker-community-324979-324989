# Habit Buddy DB schema verification (PostgreSQL)

This file documents the schema verification for the `habit_buddy_database` container.

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

## Spot checks for backend query patterns

### users columns

Verified columns via `information_schema.columns`:

- `id (uuid, not null)`
- `email (text, not null)`
- `password_hash (text, nullable)`
- `display_name (text, not null)`
- `avatar_url (text, nullable)`
- `bio (text, nullable)`
- `timezone (text, nullable)`
- `created_at (timestamptz, not null)`
- `updated_at (timestamptz, not null)`

This supports typical auth/profile queries by `email` and primary key `id`.

### habits foreign key

Verified constraint:

- `habits_user_id_fkey` → `users(id)` with `ON DELETE CASCADE`

## Notes / adjustments

- No schema changes were required during this verification pass; all required tables, indexes, and seed data were present.
- Port discrepancy note for later integration task: runtime container list mentions `5001`, while `db_connection.txt` and verified server port is `5000`. This should be handled in the integration step.

## How to re-run these checks

Run each statement as a single `psql -c "..."` command, for example:

- `psql postgresql://appuser:dbuser123@localhost:5000/myapp -c "SELECT tablename FROM pg_tables WHERE schemaname='public' ORDER BY tablename;"`
- `psql postgresql://appuser:dbuser123@localhost:5000/myapp -c "SELECT indexname, tablename FROM pg_indexes WHERE schemaname='public' ORDER BY tablename, indexname;"`
- `psql postgresql://appuser:dbuser123@localhost:5000/myapp -c "SELECT COUNT(*) FROM users;"`
