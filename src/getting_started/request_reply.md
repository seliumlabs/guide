# Request/Reply (RPC pattern)

In the previous chapter we discussed our first steps with the publish/subscribe pattern. In this
chapter we will explore Selium's other major communication pattern - request/reply (otherwise known
as RPC).

Request/reply is useful in the same way that a REST API is. For instance, you can use Selium to
replace your internal APIs:

```rust
#[derive(Deserialize)]
enum Request {
    LookupCustomer(u64),
    // Other requests
}

async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let connection = selium::custom()
        ...

    // Setup a database connection to demonstrate state sharing
    let mut db = create_db()?;

    let mut replier = connection
        .replier("/customers/api") // choose a topic to serve your API on
        .with_request_decoder(StringCodec) // let's use a StringCodec to interface with JSON clients
        .with_request_encoder(StringCodec) // let's use a StringCodec to interface with JSON clients
        .with_handler(move |json| {
            let req: Request = serde_json::from_str(&json)?;
            match req {
                Request::LookupCustomer(id) => lookup_customer(&mut db, r.client_id),
            }
        })
        .open()
        .await?;

    replier.listen().await?;

    Ok(())
}

async fn lookup_customer(db: &mut Database, id: u64) -> Result<String> {
    // Lookup request...

    // Reply with a JSON response
    Ok(r#"
        {
            "name": "John Doe",
            "age": 43,
            "phones": [
                "+44 1234567",
                "+44 2345678"
            ]
        }"#)
}
```

And now for the requestor:

```rust
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let connection = selium::custom()
        ...

    let requestor = connection
        .requestor("/some/endpoint")
        .with_request_encoder(StringCodec)
        .with_reply_decoder(StringCodec)
        .with_request_timeout(Duration::from_secs(1))?
        .open()
        .await?;

    let customer = requestor.request(r#"{"LookupCustomer": 85028}"#).await.unwrap();

    Ok(())
}
```

### Benefits over HTTP

1. No web servers
2. Built in mutual TLS
3. Ability to exchange pure Rust types over the wire
4. Transport much larger amounts of data, much faster
5. Multiple messaging patterns at your fingertips
6. 100% Rust toolchain

## Next Steps

Nice work, we've got all kinds of data flowing! Now it's time to learn about [reusing Selium connections.](./reusing_connections.md)
