# `Stream` and `Sink` traits

If you've ever used Rust's `futures` or `tokio` crates, you'll almost certainly be aware
of the [`Stream`](https://docs.rs/futures/latest/futures/stream/trait.Stream.html) and
[`Sink`](https://docs.rs/futures/latest/futures/sink/trait.Sink.html) traits. These
traits govern the sending and receiving of messages between components, and are arguably
a de facto standard in Rust.

> If you've never used these traits before, don't worry! Not only are they quite
> intuitive, but the [`futures` crate](https://docs.rs/futures/latest/futures/index.html)
> has great documentation explaining how they work.

The good news for Selium users, is that Selium's `Publisher` and `Subscriber` both
implement `Sink` and `Stream` respectively. Thus they should fit hand in glove with your
existing streaming applications.

You can also make use of the `futures` crate's
[`StreamExt`](https://docs.rs/futures/latest/futures/stream/trait.StreamExt.html) and
[`SinkExt`](https://docs.rs/futures/latest/futures/sink/trait.SinkExt.html) extensions.
These traits are implemented by default on any `Stream`/`Sink` implementations (like
Selium's) and contain lots of helpful tools.

Let's go back to our example from previous chapters and see what we can do with `SinkExt`
on a `Publisher`:

```rust
use futures::{future, stream, SinkExt, Stream, StreamExt};
use selium::{prelude::*, std::codecs::StringCodec};

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let connection = selium::custom()
        .endpoint("127.0.0.1:7001")
        .with_certificate_authority("certs/client/ca.der")? // your Selium cert authority
        .with_cert_and_key(
            "certs/client/localhost.der",
            "certs/client/localhost.key.der",
        )? // your client certificates
        .connect() // your Selium server's address
        .await?;

    let publisher = connection
        .publisher("/some/topic") // choose a topic to group similar messages together
        .with_encoder(StringCodec) // allows you to exchange string messages between clients
        .open() // opens a new stream for sending data
        .await?;

    let mut sink = publisher.with(|item: String| {
        if item.contains('@') {
            future::ok(item)
        } else {
            future::ok("default_email@company.org".into())
        }
    });
    sink.send_all(&mut some_stream()).await?;

    Ok(())
}

fn some_stream() -> impl Stream<Item = Result<String, Box<dyn std::error::Error>>> {
    stream::iter(vec![
        "hello@world.com".into(),
        "some@example.net".into(),
        "notanemail.com".into(),
    ])
    .map(|email| Ok(email))
}
```

In this example, we take a stream of email addresses and optionally map invalid email addresses to
a default email address. Of course, this is not a very useful example, but the Rust ecosystem is
_full_ of `Stream`s and `Sink`s. Selium's native support for these traits allows you to slot Selium
client into your libraries and applications with ease.

Next, let's get to grips with [codecs](./codecs.md).
