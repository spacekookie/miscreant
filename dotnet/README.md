# miscreant.net [![Build Status][build-image]][build-link] [![MIT licensed][license-shield]][license-link] [![Gitter Chat][gitter-image]][gitter-link]

[build-image]: https://secure.travis-ci.org/miscreant/miscreant.svg?branch=master
[build-link]: https://travis-ci.org/miscreant/miscreant
[license-shield]: https://img.shields.io/badge/license-MIT-blue.svg
[license-link]: https://github.com/miscreant/miscreant/blob/master/LICENSE.txt
[gitter-image]: https://badges.gitter.im/badge.svg
[gitter-link]: https://gitter.im/miscreant/Lobby

> The best crypto you've never heard of, brought to you by [Phil Rogaway]

C# implementation of **Miscreant**: Advanced symmetric encryption library
which provides the [AES-SIV] ([RFC 5297]), [AES-PMAC-SIV], and [STREAM]
constructions. These algorithms are easy-to-use (or rather, hard-to-misuse)
and support encryption of individual messages or message streams.

[AES-SIV]: https://github.com/miscreant/miscreant/wiki/AES-SIV
[RFC 5297]: https://tools.ietf.org/html/rfc5297
[AES-PMAC-SIV]: https://github.com/miscreant/miscreant/wiki/AES-PMAC-SIV
[STREAM]: https://github.com/miscreant/miscreant/wiki/STREAM

**AES-SIV** provides [nonce-reuse misuse-resistance] (NRMR): accidentally
reusing a nonce with this construction is not a security catastrophe,
unlike it is with more popular AES encryption modes like [AES-GCM].
With **AES-SIV**, the worst outcome of reusing a nonce is an attacker
can see you've sent the same plaintext twice, as opposed to almost all other
AES modes where it can facilitate [chosen ciphertext attacks] and/or
full plaintext recovery.

For more information, see the [toplevel README.md].

[nonce-reuse misuse-resistance]: https://github.com/miscreant/miscreant/wiki/Nonce-Reuse-Misuse-Resistance
[AES-GCM]: https://en.wikipedia.org/wiki/Galois/Counter_Mode
[chosen ciphertext attacks]: https://en.wikipedia.org/wiki/Chosen-ciphertext_attack
[toplevel README.md]: https://github.com/miscreant/miscreant/blob/master/README.md

## Help and Discussion

Have questions? Want to suggest a feature or change?

* [Gitter]: web-based chat about miscreant projects including **miscreant.net**
* [Google Group]: join via web or email ([miscreant-crypto+subscribe@googlegroups.com])

[Gitter]: https://gitter.im/miscreant/Lobby
[Google Group]: https://groups.google.com/forum/#!forum/miscreant-crypto
[miscreant-crypto+subscribe@googlegroups.com]: mailto:miscreant-crypto+subscribe@googlegroups.com?subject=subscribe

## Security Notice

This library has not undergone a thorough security audit. Use it at your own risk.

## Requirements

This library is targeting .NET Standard 2.0. It was tested on .NET Framework 4.7.1
and .NET Core 2.1.2, but it should also work on any of the following platforms:

- .NET Framework 4.6.1
- .NET Core 2.0
- Mono 5.4
- Xamarin.iOS 10.14
- Xamarin.Mac 3.8
- Xamarin.Android 7.5
- Universal Windows Platform 10.0.16299

## API

The `Miscreant.Aead` class provides the main interface to the **AES-SIV** misuse
resistant authenticated encryption function.

```csharp
public static Aead CreateAesCmacSiv(byte[] key)
public static Aead CreateAesPmacSiv(byte[] key)
```

To make a new AES-SIV instance that uses CMAC, call the
`CreateAesCmacSiv` method with a 32-byte or 64-byte key.
To make a new AES-SIV instance that uses PMAC, call the
`CreateAesPmacSiv` method with a 32-byte or 64-byte key.
Note that these keys are twice the size of what you might
be expecting (AES-SIV uses two AES keys).

You can generate random 32-byte or 64-byte keys using the static
`Aead.GenerateKey256` or `Aead.GenerateKey512` methods:

```csharp
var key = Aead.GenerateKey256();
var siv = Aead.CreateAesCmacSiv(key);
```

#### Encryption (Seal)

```csharp
public byte[] Seal(byte[] plaintext, byte[] nonce = null, byte[] data = null)
```

The `Seal` method encrypts a message along with a set of *associated data*
which acts as message headers.

It's recommended to include a unique "nonce" value with each message. This
prevents those who may be observing your ciphertexts from being able to tell
if you encrypted the same message twice. However, unlike other cryptographic
algorithms where using a nonce has catastrophic security implications such as
key recovery, reusing a nonce with AES-SIV only leaks repeated ciphertexts to
attackers.

#### Decryption (Open)

```csharp
public byte[] Open(byte[] ciphertext, byte[] nonce = null, byte[] data = null)
```

The `Open` method decrypts a ciphertext with the given key.

#### Example

```csharp
// Plaintext to encrypt.
var plaintext = "I'm cooking MC's like a pound of bacon";

// Create a 32-byte key.
var key = Aead.GenerateKey256();

// Create a 16-byte nonce (optional).
var nonce = Aead.GenerateNonce(16);

// Create a new AEAD instance using the AES-CMAC-SIV
// algorithm. It implements the IDisposable interface,
// so it's best to create it inside using statement.
using (var aead = Aead.CreateAesCmacSiv(key))
{
  // If the message is string, convert it to byte array first.
  var bytes = Encoding.UTF8.GetBytes(plaintext);

  // Encrypt the message.
  var ciphertext = aead.Seal(bytes, nonce);

  // To decrypt the message, call the Open method with the
  // ciphertext and the same nonce that you generated previously.
  bytes = aead.Open(ciphertext, nonce);

  // If the message was originally string,
  // convert if from byte array to string.
  plaintext = Encoding.UTF8.GetString(bytes);

  // Print the decrypted message to the standard output.
  Console.WriteLine(plaintext);
}
```

## Code of Conduct

We abide by the [Contributor Covenant][cc] and ask that you do as well.

For more information, please see [CODE_OF_CONDUCT.md].

[cc]: https://contributor-covenant.org
[CODE_OF_CONDUCT.md]: https://github.com/miscreant/miscreant/blob/master/CODE_OF_CONDUCT.md

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/miscreant/miscreant

## Copyright

Copyright (c) 2017 [The Miscreant Developers][AUTHORS].
See [LICENSE.txt] for further details.

[AUTHORS]: https://github.com/miscreant/miscreant/blob/master/AUTHORS.md
[LICENSE.txt]: https://github.com/miscreant/miscreant/blob/master/LICENSE.txt