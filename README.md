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

## 06 - Rust code to fetch and display posts

In src/bin/show_posts.rs:

```rust
extern crate diesel_demo;
extern crate diesel;

use self::diesel_demo::*;
use self::models::*;
use self::diesel::prelude::*;

fn main() {
    // import aliases so we can write:
    // - `posts` instead of `posts::table`
    // - `published` instead of `posts::published`
    use diesel_demo::schema::posts::dsl::*;

    let connection = establish_connection();
    let results = posts.filter(published.eq(true))
        .limit(5)
        .load::<Post>(&connection)
        .expect("Error loading posts");

    println!("Displaying {} posts", results.len());
    for post in results {
        println!("{}", post.title);
        println!("----------\n");
        println!("{}", post.body);
    }
}
```

```sh
cargo run -q --bin show_posts
# Displaying 0 posts
```

## 07 - Rust code to create a post

In models.rs:

```rust
use super::schema::posts;

#[derive(Insertable)]
#[table_name="posts"]
pub struct NewPost<'a> {
    pub title: &'a str,
    pub body: &'a str,
}
```

In lib.rs:

```rust
use self::models::{Post, NewPost};

pub fn create_post<'a>(conn: &PgConnection, title: &'a str, body: &'a str) -> Post {
    use schema::posts;

    let new_post = NewPost {
        title,
        body,
    };

    diesel::insert_into(posts::table)
        .values(&new_post)
        .get_result(conn)
        .expect("Error saving new post")
}
```

In bin/write_post.rs:

```rust
extern crate diesel_demo;
extern crate diesel;

use self::diesel_demo::*;
use std::io::{stdin, Read};

fn main() {
    let connection = establish_connection();

    println!("What would you like your title to be?");
    let mut title = String::new();
    stdin().read_line(&mut title).unwrap();
    let title = &title[..(title.len() - 1)]; // Drop the newline character
    println!("\nOk! Let's write {} (Press {} when finished)\n", title, EOF);
    let mut body = String::new();
    stdin().read_to_string(&mut body).unwrap();

    let post = create_post(&connection, title, &body);
    println!("\nSaved draft {} with id {}", title, post.id);
}

#[cfg(not(windows))]
const EOF: &'static str = "CTRL+D";

#[cfg(windows)]
const EOF: &'static str = "CTRL+Z";
```

```sh
cargo run -q --bin write_post
# What would you like your title to be?
# first

# Ok! Let's write first (Press CTRL+D when finished)

# tentative
# ^D
# Saved draft first with id 1
```
