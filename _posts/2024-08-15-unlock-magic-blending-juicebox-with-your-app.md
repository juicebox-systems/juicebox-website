---
layout: post
title:  "Unlock Magic: Blending Juicebox with your app"
description: "Incorporate the Juicebox SDK into your applications using CocoaPods, Maven, or npm, and test it with a local realm setup."
date:   2024-08-15 15:00:00 -0700
author-name: Nora Trapp
author-github: imperiopolis
---
Whether you’re an experienced developer or just beginning your journey in cryptography, our [SDK](https://github.com/juicebox-systems/juicebox-sdk) makes it easy to get started with the [Juicebox protocol](/assets/whitepapers/juiceboxprotocol_revision7_20230807.pdf). High security, user-friendly encryption key recovery has never been more accessible.

By the time you're finished reading this post, you should have a fully functional local environment ready to experiment with and build secure applications using Juicebox's magical key recovery features. In this guide, I'll specifically walk you through the process of integrating the Juicebox SDK into your application, running a Juicebox realm locally on your development machine, and testing the integration from end-to-end.
<!--more-->
## Prerequisites

Before diving in, make sure you have the following installed or available on your machine:

- **Go** (for building and running the Juicebox software realm)
- **Git** (for cloning repositories)
- A package manager depending on the language you're working with:
  - **CocoaPods** (for iOS)
  - **Maven** (for Android)
  - **npm** (for JavaScript/TypeScript)

Many of these tools are likely things you're already familiar with, or that came with your IDE!

If you haven’t installed these tools yet, you can find installation guides on their respective websites.

## Running a realm

Before we dive into your application, we need to get your development environment prepared with a realm that your app can talk to for testing. While there are a number of ways to run a realm for development, the fastest way to set this up is using our [software realm](https://github.com/juicebox-systems/juicebox-software-realm), so let's begin by cloning that repository.

```bash
git clone https://github.com/juicebox-systems/juicebox-software-realm
cd juicebox-software-realm
```

The software realm is written in Go, and we'll use it now to prepare a binary you can easily use in the future any time you need to run a test realm.

```bash
go build ./cmd/jb-sw-realm
```

At this point, a binary should be available in your current working directory called `jb-sw-realm`. If you run it with the `--help` option, you'll see there's a lot of ways in which it can be configured, as this realm can actually be used in production on AWS and GCP. However, for now, there are a few particular flags that you should pay attention to.

* `-id` – A 16-byte hex string identifying this realm. If you run the binary without specifying this flag, a random id will be generated. You'll need this `id` when configuring your application to talk to the realm.
* `-port` – The local port that you want the realm to communicate on. By default, this will be `8080`, but change it to fit your environment.

Additionally, you'll also want to set the `TENANT_SECRETS` environment variable, which defines a key used to validate your app's authentication with the server. We'll use the `Edwards25519` private / public key pair as follows for a tenant named `acme` for now.

```bash
private key: 302e020100300506032b657004220420565c306201d53618ceebe8b5e8c929e156e1b7473fbc0ae450542a33e8f69546
public key: 302a300506032b65700321009e8a49da843b2e4ab0984e4aee7cb9f656e19bfc1c698b1be3c89af77b258f50
```

So, putting that all together, the command to start your realm would look something like:
```bash
ACME_SECRET_1='{
  "data": "302a300506032b65700321009e8a49da843b2e4ab0984e4aee7cb9f656e19bfc1c698b1be3c89af77b258f50",
  "encoding": "Hex",
  "algorithm": "Edwards25519"
}' \
TENANT_SECRETS='{
  "acme": {
    "1": "'$(echo $ACME_SECRET_1 | tr -d '\n' | sed 's/"/\\"/g')'"
  }
}' \
./jb-sw-realm -id "000102030405060708090a0b0c0d0e0f"
```

Upon running it, you should see the software realm start up successfully with an output like:
```bash
Realm ID: 000102030405060708090a0b0c0d0e0f

Established connection to secrets manager.
Established connection to record store.
Established connection to pub/sub system.

⇨ http server started on [::]:8080
```

_**Note:**_ We've configured this realm to run with _memory_ storage of secrets. This means that each time the realm restarts, any secrets you store will be lost. You can explore the `-provider` option if you need a more permanent setup for your testing!

For ease of future testing, I recommend also making the `jb-sw-realm` binary accessible from anywhere on your system by installing it in a directory that's included in your system's `$PATH` (e.g., `/usr/local/bin` on macOS/Linux):

```bash
sudo mv jb-sw-realm /usr/local/bin/
```

Then, you can start a realm from anywhere just by running `jb-sw-realm` with the parameters we defined above.

## Installing the Juicebox SDK

With the software realm running locally, you can now install the Juicebox SDK in your app. Depending on your platform, you'll install the SDK using CocoaPods, Gradle, or npm.

### iOS with CocoaPods

Add the following line to your `Podfile`:

```ruby
pod 'JuiceboxSDK'
```

Then, run:

```bash
pod install
```

### Android with Gradle

Add the following dependency to your `build.gradle` file:

```gradle
repositories {
  mavenCentral()
}

dependencies {
  implementation 'xyz.juicebox:sdk:0.3.2'
}
```

Sync your project to download the SDK.

### JavaScript/TypeScript with npm

Install the SDK via npm:

```bash
npm install juicebox-sdk
```

## Storing your first Secret in Juicebox

With the SDK installed and the `jb-sw-realm` server running, you can now start testing the SDK in your preferred programming language.

The following outlines how to get setup and perform basic `register` and `recover` operations. Keep in mind, these are just examples to get you going, from here you'll want to hook up your user interface so the user can enter their own PIN and potentially even their own secret!

These steps are broken down based on all the platforms the Juicebox SDK supports, so you may want to jump ahead to [Swift](#swift), [Kotlin](#kotlin), or [TypeScript](#typescript) – whichever is relevant for your app.

### Swift

We'll start by instantiating an `AuthTokenGenerator` with the Edwards25519 private key we specified above.

```swift
import JuiceboxSdk

let generator = AuthTokenGenerator(json: """
  {
    "key": "302e020100300506032b657004220420565c306201d53618ceebe8b5e8c929e156e1b7473fbc0ae450542a33e8f69546",
    "tenant": "acme",
    "version": 1
  }
""")
```

You'll also need a `SecretId` in order to vend tokens, as auth tokens are specific to a realm and secret identifier. In general, this should be a 16-byte identifier that is unique for each user of your application, and you'll want to store it wherever your app stores its existing user record. For more advanced use cases, you could store multiple secrets per user by having a different identifier for secret A and secret B. For the purposes of this quick example, we're just using a fixed identifier.

```swift
let secretId = SecretId(string: "2102030405060708090a0b0c0d0e0f10")!
```

Then, we'll instantiate a `Client` with the parameters for our test realm.

```swift
Client.fetchAuthTokenCallback = { generator.vend(realmId: $0, secretId: secretId) }

let client = Client(
    configuration: Configuration(
        realms: [
            Realm(
                id: RealmId(string: "000102030405060708090a0b0c0d0e0f")!,
                address: URL(string: "http://0.0.0.0:8080")!
            )
        ],
        registerThreshold: 1,
        recoverThreshold: 1,
        pinHashingMode: .standard2019
    )
)
```

Once you've created a client, you can register a secret for the user by calling:

```swift
try await client.register(
    pin: "1234".data(using: .utf8)!,
    secret: "secret".data(using: .utf8)!,
    info: "info".data(using: .utf8)!,
    guesses: 5
)
```

To recover the secret you just registered, you can call:

```swift
let secret = try await client.recover(
    pin: "1234".data(using: .utf8)!,
    info: "info".data(using: .utf8)!
)
```

And when you're ready to delete any secret from the remote store, simply call:

```swift
try await client.delete()
```

### Kotlin

We'll start by instantiating an `AuthTokenGenerator` with the Edwards25519 private key we specified above.

```kotlin
import xyz.juicebox.sdk.*

val generator = AuthTokenGenerator("""
  {
    "key": "302e020100300506032b657004220420565c306201d53618ceebe8b5e8c929e156e1b7473fbc0ae450542a33e8f69546",
    "tenant": "acme",
    "version": 1
  }
""")
```

You'll also need a `SecretId` in order to vend tokens, as auth tokens are specific to a realm and secret identifier. In general, this should be a 16-byte identifier that is unique for each user of your application, and you'll want to store it wherever your app stores its existing user record. For more advanced use cases, you could store multiple secrets per user by having a different identifier for secret A and secret B. For the purposes of this quick example, we're just using a fixed identifier.

```kotlin
val secretId = SecretId("2102030405060708090a0b0c0d0e0f10")
```

Then, we'll instantiate a `Client` with the parameters for our test realm.

```kotlin
Client.fetchAuthTokenCallback = { realmId -> generator.vend(realmId, secretId) }

val client = Client(
  Configuration(
    realms = arrayOf(Realm(
      id = RealmId("000102030405060708090a0b0c0d0e0f"),
      address = "http://0.0.0.0:8080"
    )),
    registerThreshold = 1,
    recoverThreshold = 1,
    pinHashingMode = PinHashingMode.STANDARD_2019
  )
)
```

Once you've created a client, you can register a secret for the user by calling:

```kotlin
client.register("1234".toByteArray(), "secret".toByteArray(), "info".toByteArray(), 5)
```

To recover the secret you just registered, you can call:

```kotlin
val secret = String(client.recover("1234".toByteArray(), "info".toByteArray()))
```

And when you're ready to delete any secret from the remote store, simply call:

```kotlin
client.delete()
```

### TypeScript

We'll start by instantiating an `AuthTokenGenerator` with the Edwards25519 private key we specified above.

```typescript
import { Client, Configuration, AuthTokenGenerator } from 'juicebox-sdk';

const generator = new AuthTokenGenerator({
  "key": "302e020100300506032b657004220420565c306201d53618ceebe8b5e8c929e156e1b7473fbc0ae450542a33e8f69546",
  "tenant": "acme",
  "version": 1
});
```

You'll also need a `SecretId` in order to vend tokens, as auth tokens are specific to a realm and secret identifier. In general, this should be a 16-byte identifier that is unique for each user of your application, and you'll want to store it wherever your app stores its existing user record. For more advanced use cases, you could store multiple secrets per user by having a different identifier for secret A and secret B. For the purposes of this quick example, we're just using a fixed identifier.

```typescript
const secretIdString = "2102030405060708090a0b0c0d0e0f10";
```

Then, we'll instantiate a `Client` with the parameters for our test realm.

```typescript
window.JuiceboxGetAuthToken = async (realmId) => {
  const realmIdString = Buffer.from(realmId).toString('hex');
  return generator.vend(realmIdString, secretIdString);
};

const client = Client(
    new Configuration({
        realms: [
          {
            "id": "000102030405060708090a0b0c0d0e0f",
            "address": "http://0.0.0.0:8080"
          }
        ],
        register_threshold: 1,
        recover_threshold: 1,
        pin_hashing_mode: "Standard2019"
    }),
    []
)
```

Once you've created a client, you can register a secret for the user by calling:

```typescript
await client.register(encoder.encode("1234"), encoder.encode("secret"), encoder.encode("info"), 2);
```

To recover the secret you just registered, you can call:

```typescript
const secret = await client.recover(encoder.encode("1234"), encoder.encode("info"));
```

And when you're ready to delete any secret from the remote store, simply call:

```typescript
await client.delete();
```

## Conclusion and Next Steps

Congratulations! By following this guide, you've hopefully successfully set up a local environment with the Juicebox SDK, integrated it into your application, and tested registering and recovering secrets. This is just the beginning of what you can achieve with Juicebox, but the rest of the magic is up to you!

Now that you have a solid foundation, here are some recommended next steps to further enhance your implementation:

* **Learn How Juicebox Works** – Take a [deeper dive](/blog/key-to-simplicity-squeezing-the-hassle-out-of-encryption-key-recovery) into the Juicebox Protocol that's behind our realms and SDK.
* **Deploy the Software Realm to the Cloud** – We have [detailed steps](https://github.com/juicebox-systems/juicebox-software-realm?tab=readme-ov-file#running-a-realm) that make it easy to get a production ready software realm running in GCP or AWS.
* **Explore Hardware Security Modules (HSMs)** – For even higher security, consider [setting up a hardware realm](/blog/running-a-hsm-realm) backed by HSMs. HSMs provide an added layer of protection by securely managing cryptographic keys in dedicated hardware devices.
* **Contribute to Juicebox** – If you’re passionate about open-source development, consider contributing to the [Juicebox project](https://github.com/juicebox-systems). Whether it’s improving documentation, fixing bugs, or adding new features, your contributions can help make Juicebox even better.

Happy coding, and welcome to the future of key recovery!
