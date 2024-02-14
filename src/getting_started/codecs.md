# Codecs
Looking back at the previous chapters, you probably noticed lines like these ones:

```rust
let mut publisher = connection
    ...
    .with_encoder(StringCodec) // allows you to exchange string messages between clients

let mut subscriber = connection
    ...
    .with_decoder(StringCodec) // use the same codec as the publisher
```

If you've not worked with data streaming before, codecs might seem like a very strange
concept indeed. However the answer is quite simple. The term _codecs_ is a portmanteau of
_encoder_ and _decoder_. Thus a codec is simply a structure that can _encode_ frames of
data on one side of a connection, and _decode_ them on the other.

In the example above, we use `StringCodec`, which does exactly what it says on the tin -
it _encodes_ strings to bytes and then _decodes_ those bytes back to strings.

## Why do we need codecs?

We need codecs because at the network level, computers only support sending raw bytes.
However most data we work with are not raw bytes - they're strings, integers, booleans,
enumerators, structures etc. In order to abstract away the pain of converting these types
to bytes and back again, we use a codec that knows how to do it for us.

## Can I send more than just strings?

Yep! With the help of the `serde` crate, we support sending all manner of Rust types.
In the example below, we use the `BincodeCodec` to send a stream of `StockEvent`s.

You'll need to install `serde` with the _derive_ feature to follow this example:
```bash
$ cargo add -F derive serde
```

```rust
use futures::SinkExt;
use selium::{prelude::*, std::codecs::BincodeCodec};
use serde::{Deserialize, Serialize};

#[derive(Debug, Serialize, Deserialize)]
struct StockEvent {
    ticker: String,
    change: f64,
}

impl StockEvent {
    pub fn new(ticker: &str, change: f64) -> Self {
        Self {
            ticker: ticker.to_owned(),
            change,
        }
    }
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let connection = selium::custom()
        .endpoint("127.0.0.1:7001")
        .with_certificate_authority("certs/client/ca.der")?
        .with_cert_and_key(
            "certs/client/localhost.der",
            "certs/client/localhost.key.der",
        )?
        .connect()
        .await?;

    let mut publisher = connection
        .publisher("/some/topic")
        .with_encoder(BincodeCodec::default())
        .open()
        .await?;

    publisher.send(StockEvent::new("INTC", -9.0)).await?;
    publisher.finish().await?;

    Ok(())
}
```

## Can I build my own codec?

Yes, and you can use third party codecs like protocol buffers, SBE etc. We'll have a chapter on
this coming soon.
