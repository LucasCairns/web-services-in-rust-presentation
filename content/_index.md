+++
title = "My presentation"
outputs = ["Reveal"]
+++

# Are we web yet?

Writing web services in Rust

---

## What is Rust?

- general purpose programming language
- announced by Mozilla in 2010
- "C" style syntax
- features you would expect from a modern language
- rising popularity amongst developers

---

## Performance

- compiled language with no run time
- no garbage collector

---

## Reliability

- statically typed
- type system ensures errors are handled 
- ownership guarantees both memory and thread safety

---

## Productivity

- compiler provides useful error messages
- best in class package manager and build tool
- extensive documentation

---

## The Rust language

the bits I really like

---

## Error handling

{{< highlight rust "linenos=inline" >}}
async fn init_db() -> Result<PgPool, Error> {
    let database_url = env::var("DATABASE_URL")?;

    match PgPoolOptions::new()
        .connect(&database_url)
        .await {
            Ok(pool) => {
                sqlx::migrate!("db/migrations")
                    .run(&pool)
                    .await?;
                Ok(pool)
            }, 
            Err(e) => Err(e.into())
        }
}
{{< / highlight >}}

---

{{% section %}}

## Trait system

Derivable traits

{{< highlight rust "linenos=inline" >}}
#[derive(Debug, Deserialize, Serialize)]
struct Person {
    first_name: String,
    family_name: String,
    date_of_birth: Date,
}
{{< / highlight >}}

---

Implementing a trait

{{< highlight rust "linenos=inline" >}}
impl From<Claims> for ReadUser {
    fn from(claims: Claims) -> Self {
        ReadUser {
            username: claims.sub,
        }
    }
}

// Implementing this would allow for the following
//
// let user = ReadUser::from(claims);
// let user: ReadUser = claims.into();
// fn some_function(user: dyn Into<ReadUser>) {}
{{< / highlight >}}

{{% /section %}}

---

{{% section %}}

## Pattern matching

{{< highlight rust "linenos=inline,hl_lines=3-6" >}}
impl IntoResponse for ApiError {
    fn into_response(self) -> axum::response::Response {
        let validation_errors = match &self {
            ApiError::ValidationError(e) => Some(e),
            _ => None,
        };

        (
            self.status_code(),
            Json(ErrorResponse {
                message: &self,
                errors: validation_errors,
            }),
        ).into_response()
    }
}
{{< / highlight >}}

---

They're exhaustive

{{< highlight rust "linenos=inline" >}}
fn handle_error(error: sqlx::Error) {
    match error {
        sqlx::Error::Configuration(_) => todo!(),
        sqlx::Error::Database(_) => todo!(),
        sqlx::Error::Io(_) => todo!(),
        sqlx::Error::Tls(_) => todo!(),
        sqlx::Error::Protocol(_) => todo!(),
        sqlx::Error::RowNotFound => todo!(),
        sqlx::Error::TypeNotFound { type_name } => todo!(),
        sqlx::Error::ColumnIndexOutOfBounds { index, len } => todo!(),
        sqlx::Error::ColumnNotFound(_) => todo!(),
        sqlx::Error::ColumnDecode { index, source } => todo!(),
        sqlx::Error::Decode(_) => todo!(),
        sqlx::Error::PoolTimedOut => todo!(),
        sqlx::Error::PoolClosed => todo!(),
        sqlx::Error::WorkerCrashed => todo!(),
        sqlx::Error::Migrate(_) => todo!(),
        _ => todo!(),
    }
}
{{< / highlight >}}

{{% /section %}}

---

## Rust for web services

I built a thing and this is what I learned

---

{{% section %}}

## axum

- web framework
- part of the Tokio ecosystem
- uses Hyper HTTP libraries
- really fast
- minimal boilerplate

---

### What it looks like

{{< highlight rust "linenos=inline" >}}
#[tokio::main]
async fn main() {
    let app = Router::new().route("/", get(say_hello));
    
    let addr = SocketAddr::from(([127, 0, 0, 1], 8080));
    axum::Server::bind(&addr)
        .serve(app.into_make_service())
        .await
        .unwrap();
}

async fn say_hello() -> Html<&'static str> {
    Html("Hello, World!")
}
{{< / highlight >}}

---

### Other web frameworks

- Actix web
    - tried and tested
    - originally built upon the Actix library

- Rocket
    - simple use of macros for code-gen
    - good quality documentation

{{% /section %}}

---

{{% section %}}

## sqlx

- supports PostgreSQL, MySQL, SQLite, and MSSQL
- async
- it's not an ORM
- queries are checked at compile time

---

## Executing queries

```rust
let person = sqlx::query_as!(
    Person,
    r#"
        INSERT INTO person (first_name, family_name, date_of_birth)
        VALUES ($1, $2, $3)
        RETURNING uuid AS id, created, last_edited, first_name, family_name, date_of_birth;
    "#,
    request.first_name,
    request.family_name,
    request.date_of_birth
)
.fetch_one(&db)
.await?;
```

---

## Applying migrations

Programmatically

{{< highlight rust "linenos=inline" >}}
sqlx::migrate!("db/migrations").run(&pool).await?;
{{< / highlight >}}

Using the CLI

{{< highlight sh "linenos=inline" >}}
sqlx migrate run --source=db/migrations
{{< / highlight >}}

---

### If you want to use an ORM

- Diesel
    - a mature library
    - robust compile time type checking

- SeaOrm
    - async
    - built upon sqlx
    - really good tooling

{{% /section %}}

---

{{% section %}}

## serde and validator

- Serde
    - JSON serialization and deserialization
    - derive macros make things quick and easy
- Validator
    - macro based validation for DTOs
    - a lot of validation out the box
    - extendable

--- 

## Building a DTO

{{< highlight rust "linenos=inline" >}}
#[derive(Debug, Validate, Deserialize)]
pub struct NewPerson {
    #[validate(length(min = 1, max = 64))]
    first_name: String,
    #[validate(length(min = 1, max = 64))]
    family_name: String,
    #[validate(custom = "date_not_in_future")]
    date_of_birth: Date,
}
{{< / highlight >}}

---

## Running the validation

{{< highlight rust "linenos=inline, hl_lines=5" >}}
async fn create_person(
    db: Extension<PgPool>,
    Json(request): Json<NewPerson>,
) -> Result<(StatusCode, Json<Person>), ApiError> {
    request.validate()?;
    // ...
}
{{< / highlight >}}

{{% /section %}}

---

## So what did I not like?

- maturity of some of the libraries
- documentation is generally good, but examples can be lacking
- the borrow checker and lifetimes still kick my ass from time to time

---

## Useful links

- [the official Rust website](https://www.rust-lang.org/)
- [the Rust book](https://doc.rust-lang.org/book/)
- [are we web yet?](https://www.arewewebyet.org/)
- [my rust web service](https://github.com/LucasCairns/rust-web-app)

---

# Questions? 🤔
