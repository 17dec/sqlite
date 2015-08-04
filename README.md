# SQLite [![Version][version-img]][version-url] [![Status][status-img]][status-url]

The package provides an interface to [SQLite][1].

## [Documentation][doc]

## Example

Open a connection, create a table, and insert a couple of rows:

```rust
let connection = sqlite::open(":memory:").unwrap();

connection.execute("
    CREATE TABLE users (name TEXT, age INTEGER);
    INSERT INTO users (name, age) VALUES ('Alice', 42);
    INSERT INTO users (name, age) VALUES ('Bob', 69);
").unwrap();
```

Select a row from the table:

```rust
connection.iterate("SELECT * FROM users WHERE age > 50", |pairs| {
    for &(column, value) in pairs.iter() {
        println!("{} = {}", column, value.unwrap());
    }
    true
}).unwrap();
```

The same query using a prepared statement:

```rust
use sqlite::State;

let mut statement = connection.prepare("
    SELECT * FROM users WHERE age > ?
").unwrap();

statement.bind(1, 50).unwrap();

while let State::Row = statement.next().unwrap() {
    println!("name = {}", statement.read::<String>(0).unwrap());
    println!("age = {}", statement.read::<i64>(1).unwrap());
}
```

The same query example using a cursor, which is a wrapper over a prepared
statement:

```rust
use sqlite::Value;

let mut cursor = connection.prepare("
    SELECT * FROM users WHERE age > ?
").unwrap().cursor();

cursor.bind(&[Value::Integer(50)]).unwrap();

while let Some(row) = cursor.next().unwrap() {
    match (&row[0], &row[1]) {
        (&Value::String(ref name), &Value::Integer(age)) => {
            println!("name = {}", name);
            println!("age = {}", age);
        },
        _ => unreachable!(),
    }
}
```

## Contributing

1. Fork the project.
2. Implement your idea.
3. Open a pull request.

[1]: https://www.sqlite.org

[version-img]: http://stainless-steel.github.io/images/crates.svg
[version-url]: https://crates.io/crates/sqlite
[status-img]: https://travis-ci.org/stainless-steel/sqlite.svg?branch=master
[status-url]: https://travis-ci.org/stainless-steel/sqlite
[doc]: https://stainless-steel.github.io/sqlite
