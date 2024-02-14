# Selium Cloud

Selium Cloud (Beta) consists of a managed Selium Server, authentication, certificate
management, dashboard and technical support. If you want a hands-free, production-ready
solution for all of your software comms, backed by the people that made Selium, this is
for you.

## Getting Started

#### 1. Sign up
To get started with Selium Cloud, you'll [need an account!](https://www.selium.com/#CTA)
Selium Cloud is free for life, and super affordable as you grow with us.

#### 2. Create your first certificate
Once you've got your account, login to the Selium Cloud dashboard at
[cloud.selium.io](https://cloud.selium.io). From here, you can create TLS certificates
for each client on your account.

After logging in, you should see a screen like this:

<img width="800" alt="Dashboard" src="https://github.com/seliumlabs/selium/assets/3638076/3ad412cf-7d8b-4f02-b632-e6728c7c9ce6">

To create a certificate, fill in a name for your client, e.g. "web1.example.com", then
click the submit button. After a moment you should see a dialog box like this:

<img width="800" alt="Add client" src="https://github.com/seliumlabs/selium/assets/3638076/cf2ab47c-86d3-4f4f-a94c-8b26ca425303">

**Make sure you download the private key as you cannot download it again!**

Add the public and private keys to your project and you're ready to start using Selium.

## Basic Usage

Selium Cloud's API feels just like running your own Selium server. The main difference is
when establishing a connection to the server. Instead of using `selium::custom()` to
connect to your own server, use `selium::cloud()` to connect to the cloud:

```rust
let connection = selium::cloud()
    .with_cert_and_key(
        "./web1.example.com.der",
        "./web1.example.com.key.der",
    )?
    .connect()
    .await?;
```

Note that when using `selium::cloud()`, you won't need to specify a CA path or endpoint.
These details are baked into the Selium client for your convenience.

#### Namespaces

Each Selium Cloud account is linked to a unique _namespace_. This is used to distinguish
your data from other accounts, and is linked to every certificate you create. You can find
your namespace on the Selium Cloud dashboard:

<img width="800" alt="Namespace" src="https://github.com/seliumlabs/selium/assets/3638076/f5b9761b-da92-4d5c-9611-471e92ddd20c">

To use your namespace, simply prepend it to every topic name. For example, to publish the
topic "retail-transactions", you would code the following:

```rust
let mut publisher = connection
    .publisher("/example/retail-transactions")
    ...
    .await?;
```

### IMPORTANT NOTE!

**You must compile your code with the `--release` flag for both testing and production
use. This is so the Selium client knows to use the _production_ Certificate Authority!**

If you don't do this, you will not be able to connect to Selium.

> N.B. We know this isn't ideal, and likely we'll be removing this restriction in future
> versions. We'd love your feedback on this too!
