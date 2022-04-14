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
