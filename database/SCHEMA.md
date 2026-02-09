# Database schema (AI Idea Generator)

This database schema supports:
- Storing generated idea sets (`ideas`)
- Saving/bookmarking idea sets (`saved_items`)
- Creating shareable links/tokens for idea sets (`share_tokens`)

## Connection (source of truth)

Use the command in:

- `database/db_connection.txt`

Example:

```bash
PGCMD="$(cat db_connection.txt)"
$PGCMD -c "SELECT 1;"
```

## Applied DDL (executed one statement at a time)

### 1) UUID support

```bash
PGCMD="$(cat db_connection.txt)"
$PGCMD -c "CREATE EXTENSION IF NOT EXISTS pgcrypto;"
```

### 2) ideas

Stores the original topic/prompt and the generated ideas payload (as JSON).

```bash
PGCMD="$(cat db_connection.txt)"
$PGCMD -c "CREATE TABLE IF NOT EXISTS ideas (id uuid PRIMARY KEY DEFAULT gen_random_uuid(), topic text NOT NULL, prompt text, ideas jsonb NOT NULL, model text, created_at timestamptz NOT NULL DEFAULT now(), updated_at timestamptz NOT NULL DEFAULT now());"
```

Index:

```bash
PGCMD="$(cat db_connection.txt)"
$PGCMD -c "CREATE INDEX IF NOT EXISTS idx_ideas_created_at ON ideas(created_at DESC);"
```

### 3) saved_items

Represents user “saved” items linked to an `ideas` record.

```bash
PGCMD="$(cat db_connection.txt)"
$PGCMD -c "CREATE TABLE IF NOT EXISTS saved_items (id uuid PRIMARY KEY DEFAULT gen_random_uuid(), idea_id uuid NOT NULL REFERENCES ideas(id) ON DELETE CASCADE, title text, notes text, created_at timestamptz NOT NULL DEFAULT now());"
```

Index:

```bash
PGCMD="$(cat db_connection.txt)"
$PGCMD -c "CREATE INDEX IF NOT EXISTS idx_saved_items_idea_id ON saved_items(idea_id);"
```

### 4) share_tokens

Token-based sharing for an `ideas` record.

```bash
PGCMD="$(cat db_connection.txt)"
$PGCMD -c "CREATE TABLE IF NOT EXISTS share_tokens (id uuid PRIMARY KEY DEFAULT gen_random_uuid(), token text NOT NULL UNIQUE, idea_id uuid NOT NULL REFERENCES ideas(id) ON DELETE CASCADE, created_at timestamptz NOT NULL DEFAULT now(), expires_at timestamptz, revoked_at timestamptz);"
```

Index:

```bash
PGCMD="$(cat db_connection.txt)"
$PGCMD -c "CREATE INDEX IF NOT EXISTS idx_share_tokens_token ON share_tokens(token);"
```

### 5) updated_at trigger for ideas

Function:

```bash
PGCMD="$(cat db_connection.txt)"
$PGCMD -c "CREATE OR REPLACE FUNCTION set_updated_at() RETURNS trigger LANGUAGE plpgsql AS 'BEGIN NEW.updated_at = now(); RETURN NEW; END;';"
```

Trigger (drop + create for idempotency):

```bash
PGCMD="$(cat db_connection.txt)"
$PGCMD -c "DROP TRIGGER IF EXISTS trg_ideas_updated_at ON ideas;"
```

```bash
PGCMD="$(cat db_connection.txt)"
$PGCMD -c "CREATE TRIGGER trg_ideas_updated_at BEFORE UPDATE ON ideas FOR EACH ROW EXECUTE FUNCTION set_updated_at();"
```

## Verification

List the expected tables:

```bash
PGCMD="$(cat db_connection.txt)"
$PGCMD -c "SELECT table_name FROM information_schema.tables WHERE table_schema='public' AND table_name IN ('ideas','saved_items','share_tokens');"
```

## Seed data

No seed data was inserted (not required for schema verification).
