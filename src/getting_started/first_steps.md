# First Steps

Selium comprises a client library and a server binary. In order to use Selium, there are 3 basic
steps:
1. Create TLS certificates for your client and server
2. Run the server binary
3. Integrate the client library into your project

## Step 1 - TLS Certificates

First we need some certs to validate the client and server. Selium uses mutual TLS (_mTLS_) to
validate both parties cryptographically, making things nice and secure.

We've built a tool to make this easy, so let's install that, then mint our certs:

```bash
# Install the selium-tools CLI
$ cargo install selium-tools
# Use this CLI to create our certificates
$ selium-tools gen-certs
```

You should now have a directory in the current path called `certs/`. Inside we have certs for
the client and server, which you can move to a more convenient location if you like - the paths are
configurable in code. Both the `client/` and `server/` directories include a copy of the
certificate authority, which you'll need if you want to create more client certificates later.

## Step 2 - Start the Selium Server

The Selium server allows us to exchange messages between clients. For this example we'll grab a
copy from [crates.io](https://crates.io/crates/selium-server).

> For production deployments you can also download prebuilt binaries from
> [GitHub](https://github.com/orgs/seliumlabs/packages?repo_name=selium).

Let's start a new server with our freshly minted certs. In the same directory as your `certs/`
folder, open a new terminal and run the following commands:

```bash
# Install Selium server
$ cargo install selium-server
# Run the server
$ selium-server --bind-addr=127.0.0.1:7001
```

The `selium-server` command will not produce any output by default, but that doesn't mean it
isn't working! You can increase logging using the verbosity flag:

```bash
$ selium-server -v # Warnings only
$ selium-server -vv # Info
$ selium-server -vvv # Debug
$ selium-server -vvvv # Trace
```

## Step 3 - Implement the Selium Client

Selium Client is a composable library API for the Selium Server. Let's have a look at a minimal
example:

```rust
use futures::{SinkExt, StreamExt};
use selium::{prelude::*, std::codecs::StringCodec};

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let connection = selium::custom() // connect to your own Selium server
        .endpoint("127.0.0.1:7001") // your Selium server's address
        .with_certificate_authority("certs/client/ca.der")? // your Selium cert authority
        .with_cert_and_key(
            "certs/client/localhost.der",
            "certs/client/localhost.key.der",
        )? // your client certificates
        .connect()
        .await?;

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

    // Send a message and close the publisher
    publisher.send("Hello, world!".into()).await?;
    publisher.finish().await?;

    // Receive the message
    if let Some(Ok(message)) = subscriber.next().await {
        println!("Received message: {message}");
    }

    Ok(())
}
```

There's a lot to take in here, but for the moment let's just get this baby running!

```bash
# Create a new Cargo project
$ cargo new --bin hello-selium

# Move into our project
$ cd hello-selium

# Add the crate dependencies
$ cargo add futures
$ cargo add -F std selium
$ cargo add -F macros,rt tokio
```

Now copy and paste the code above into `hello-selium/src/main.rs`.

Now let's execute the project and start exchanging some messages! Make sure your server is still
running from the previous step.

```bash
$ cargo run
Received message: Hello, world!
```

## Next Steps

We've just setup a working Selium publish/subscribe project, but [you can also use RPC too.](./request_reply.md)
