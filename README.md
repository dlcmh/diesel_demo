# Diesel - Getting Started

http://diesel.rs/guides/getting-started.html

## 01 - Initialize new project

```
cargo new --lib diesel_demo
cd diesel_demo
```

In Cargo.toml:

```toml
diesel = { version = "1.4.4", features = ["postgres"] }
dotenv = "0.15.0"
```

## 02 - Install Diesel CLI

To install only for PostgreSQL:

```
cargo install diesel_cli --no-default-features --features postgres
```

## 03 - Set up Diesel for project

In .env:

```sh
# DATABASE_URL=postgres://username:password@localhost/diesel_demo
DATABASE_URL=postgres:///diesel_demo
```

```sh
diesel setup
# Creating migrations directory at: /Users/dlcmh/projects/rust-apps/rust-diesel-getting-started/migrations
# Creating database: diesel_demo
```

## 04 - Diesel migration to create a table

```sh
diesel migration generate create_posts
# Creating migrations/2022-04-14-015826_create_posts/up.sql
# Creating migrations/2022-04-14-015826_create_posts/down.sql
```

In up.sql:

```sql
CREATE TABLE posts (
  id SERIAL PRIMARY KEY,
  title VARCHAR NOT NULL,
  body TEXT NOT NULL,
  published BOOLEAN NOT NULL DEFAULT 'f'
)
```

In down.sql:

```sql
DROP TABLE posts
```

```sh
diesel migration run
# Running migration 2022-04-14-015826_create_posts
```
