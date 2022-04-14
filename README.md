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

## 05 - Rust code to set up DB connection and `Post` model

In src/lib.rs:

```rust
pub mod schema;
pub mod models;

// #[macro_use] required, otherwise:
// - `models.rs` will have the following error (via `pub mod models`):
//   cannot find derive macro `Queryable` in this scope
//   consider importing this derive macro:
//   diesel::Queryable
// - `schema.rs` will have the following error (via `pub mod schema`):
//   cannot find macro `table` in this scope
//   consider importing this macro:
//   diesel::table
#[macro_use]
extern crate diesel;
extern crate dotenv;

use diesel::prelude::*;
use diesel::pg::PgConnection;
use dotenv::dotenv;
use std::env;

pub fn establish_connection() -> PgConnection {
    dotenv().ok();

    let database_url = env::var("DATABASE_URL")
        .expect("DATABASE_URL must be set");
    PgConnection::establish(&database_url)
        .expect(&format!("Error connecting to {}", database_url))
}
```

In models.rs:

```rust
// #[derive(Queryable)]:
// - generates code to load a Post struct from an SQL query
// - order of fields on the Post struct must match order in `posts` table in schema.rs
#[derive(Queryable)]
pub struct Post {
    pub id: i32,
    pub title: String,
    pub body: String,
    pub published: bool,
}
```
