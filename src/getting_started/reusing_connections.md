# Reusing Connections

Selium supports reusing connections to the Selium server, otherwise known as _multiplexing_. This
allows you to maintain a single connection that publishes and/or subscribes to multiple topics
simultaneously.

Let's have another look at that code from the previous chapter:

```rust
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let connection = selium::custom()
        ...

    let mut publisher = connection
        .publisher("/some/topic") // choose a topic to group similar messages together
        .with_encoder(StringCodec) // allows you to exchange string messages between clients
        .open() // opens a new stream for sending data
        .await?;

    let mut subscriber = connection
        .subscriber("/some/topic") // subscribe to the publisher's topic
        .with_decoder(StringCodec) // use the same codec as the publisher
        .open() // opens a new stream for receiving data
        .await?;

    ...

    Ok(())
}
```

We can see that each time a new behaviour is required, we create a new stream using
`Client::publisher` and `Client::subscriber`. For each new stream, the full range of builder
options is available, uniquely for that stream.

Next, let's see how we can send/receive data more ergonomically with
[`Stream` and `Sink` traits](./streams_and_sinks.md).
