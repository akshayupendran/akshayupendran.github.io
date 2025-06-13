# OpenSSL CLI Tutorial

## Installation

Below command installs openssl:

```bash
sudo apt install openssl
```

Output:

```bash
openssl is already the newest version (3.0.13-0ubuntu3.5)
```

You can check the installed version with:

```bash
openssl version
```

Output:

```bash
OpenSSL 3.0.13 30 Jan 2024 (Library: OpenSSL 3.0.13 30 Jan 2024)
```

## Introduction

```bash
openssl <command> [options] [arguments]
```

Before we go into the details, let us first learn a bit of theory on Asymmetric Cryptography, EC and RSA.

## Asymmetric Key Cryptography

Asymmetric key cryptography is a form of cryptography where there are two keys involved - a public key and a private key.
The public key is distributed to other people but the private key is always held a secret.
There are two main forms in which asymmetric key cryptography is used: Encryption/Decryption and SignatureGeneration/Verification.
The server generates a key pair and distributes the public key to the clients.
In case of Encryption/Decryption, the clients can encrypt their transmission using the public key and the server would be the only one who is able to decrypt it as he holds the private key.
Even though the above method is logical, as its extremely time consuming, the client usually establishes a session by encrypting a symmetric key with the public key and transmits it to the server, who decrypts it using the private key and uses that symmetric key for all future communications.
Hence this article will mainly explore the generation of the key pair and encryption/decryption and signatureGeneration/Verification using RSA and EC (Elliptic Key Cryptography).
The various asymmetric key cryptography algorithms supported by OpenSSL utility are RSA, RSA-PSS, EC, X25519, X448, ED25519 and ED448.

RSA is the most common form of asymmetric key cryptography.

## OpenSSL CLI Introduction

We have a list of OpenSSL sub utilities which we can use to demonstrate the different types of asymmetric key cryptographic operations.

The main sub utilities and their respective purposes are as follows:

1. GenPKey is a sub utility to generate Private Keys.
2. RSA is a sub utility for processing RSA keys.
3. EC is a sub utility for processing EC keys.
4. DGST is a sub utility for generating a digest and also signing / verifying the digest using a private key.
5. PKeyUtl is a sub utility to sign/verify hash and also encryption/decryption of plain text.

OpenSSL has a few defaults which are as follows:

- OpenSSL assumes the keys in the arguments to be of the PEM(privacy Enhanced Mail) format if not specified.
- OpenSSL considers the keys in arguments to be private keys by default.

## PEM vs DER

DER is the preferred ASN.1 syntax for Cryptographic Keys and for identifying Schemes.
But as DER is not transmittable over a network, the DER is base64 encoded to form the PEM format.
Update DER encoding here #ToDo
Update ASN.1 encoding here #ToDo

## RSA

References:
[PKCS #1 specification -> RFC 8017](https://tools.ietf.org/html/rfc8017)

- A RSA Private key consists of:
  - version (0 unless multi prime is used in which case its 1)
  - Modulus (same length as key)
  - publicExponent
  - privateExponent (same length as key)
  - prime1
  - prime2
  - exponent1
  - exponent2
  - coefficient
  - prime n
  - exponent n
  - coefficient n
- A RSA Public key is nothing but extracting the Modulus and public exponent.
- Hence a public key consists of:
  - Modulus
  - Public Exponent

OpenSSL defaults:

- The RSA key sizes shall be a minimum of 1024 bits.
- The default size of RSA key in OpenSSL is 2048 bits.
- The default number of prime numbers used is 2.
- The maximum number of prime numbers supported is 3.
- The default public exponent is 65537.
- As the public exponent becomes smaller and smaller, the computation time becomes lesser.

## EC

Elliptic Curve public key is of the following format:

- first bit is compression
- ((n-1)/2) bits are x
- ((n-1)/2) bits are y

Structured version of ECC Keys are to be described #ToDo

## Practical Experiments

### Generating a private key (RSA)

The sub-utility used for generating a private key is ```genpkey```.

Generating a RSA Private Key in PEM format:

```sh
openssl genpkey -out private_key.pem -algorithm RSA
```

Generating a RSA 3072 bit Private Key with 3 primes and a public exponent 17 in DER format:

```sh
openssl genpkey -out private_key.der -outform DER -algorithm RSA -pkeyopt rsa_keygen_bits:3072 -pkeyopt rsa_keygen_primes:3 -pkeyopt rsa_keygen_pubexp:17
```

To confirm if our parameters have taken effect, we can run the same in -text without an output file.

```sh
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:3072 -pkeyopt rsa_keygen_primes:3 -pkeyopt rsa_keygen_pubexp:17 -text
```

### Extracting the public Key (RSA)

Prerequisites
[Generating a private key (RSA)](#generating-a-private-key-rsa)

The public key can be extracted from the private key using the ```rsa``` sub-utility.

We can extract the public key using the RSA utility.
There are two major formats for a RSA Public key.

The PEM public key format uses the header and footer lines:
-----BEGIN PUBLIC KEY-----
-----END PUBLIC KEY-----

The PEM RSAPublicKey format uses the header and footer lines:

-----BEGIN RSA PUBLIC KEY-----
-----END RSA PUBLIC KEY-----

To extract the key in the first PEM public key format:

```sh
openssl rsa -in private_key.pem -pubout -out public_key.pem
```

To extract the key in the second PEM RSAPublicKey key format:

```sh
openssl rsa -in private_key.pem -RSAPublicKey_out -out public_key.pem
```

### Generating a signature (RSA)

Prerequisites
[Generating a private key (RSA)](#generating-a-private-key-rsa)

Signature can be created either using the ```dgst``` sub util or using the ```pkeyutl``` sub utility.

Lets create a plain text:

```sh
echo "Wonderful Wolrd!" > plain_text.txt
```

There are two ways we can sign a plain text as described above.

Let us first generate the signature using the ```dgst``` sub utility.

```sh
openssl dgst -sha256 -out signature.sig -sign private_key.pem plain_text.txt
```

Let us create another signature using the second method.
For this first lets generate the SHA2-256 digest of the ```plain_text.txt```.

```sh
openssl dgst -sha256 -binary -out hash.sha256 plain_text.txt
```

Now let us sign the generated hash using ```pkeyutl```.

```sh
openssl pkeyutl -sign -in hash.sha256 -inkey private_key.pem -out signature2.sig -pkeyopt digest:sha256
```

To show that both methods produce the same signature, we can confirm without

```sh
diff signature.sig signature2.sig
```

Investigate why OpenSSL does not generate a private key of -----BEGIN RSA PRIVATE KEY----- but rather generates -----BEGIN PRIVATE KEY-----. #ToDo

### Verifying a signature (RSA)

Prerequisites
[Generating a signature (RSA)](#generating-a-signature-rsa) and [Extracting the public Key (RSA)](#extracting-the-public-key-rsa)

The verify signature is similar to [Generating a signature (RSA)](#generating-a-signature-rsa)

```sh
openssl dgst -sha256 -verify public_key.pem -signature signature.sig plain_text.txt
```

The second method is the roundabout way of calculating hash and then verifying the signature.

But this was not successful for me - hence lets update when we get the time. #ToDo

### Encrypting a file (RSA)

Prerequisites
[Extracting the public Key (RSA)](#extracting-the-public-key-rsa)

We could encrypt any plain text using the public key as follows:

```sh
openssl pkeyutl -encrypt -inkey public_key.pem -pubin -in plain_text.txt -out cipher_text.txt
```

### Decrypting a file (RSA)

Prerequisites
[Generating a private key (RSA)](#generating-a-private-key-rsa)

We could decrypt any plain text using the private key as follows:

```sh
openssl pkeyutl -decrypt -inkey private_key.pem -in cipher_text.txt -out plain_text2.txt
```

### Encrypted Private keys (RSA)

An encrypted private key is better and safer than a plain private key.

#### Generation of private Key (Encrypted RSA)

```sh
openssl genpkey -out private_key2.pem -algorithm RSA -aes-128-cbc -pass pass:hello
```

Now lets find out what exactly happens in the background.

Let us create a non encrypted private key from the encrypted one.

```sh
openssl rsa -in private_key2.pem -passin pass:hello -out private_key3.pem
```

I still don't know how to encrypt private_key3.pem to make it private_key2.pem #ToDo
I could encrypt using ```openssl enc -aes-128-cbc -e -pass pass:hello -in private_key3.pem -out private_key4.pem -a #ToDo

Hence lets fallback and try the other operations.

#### Extracting the Public key (Encrypted RSA)

```sh
openssl rsa -in private_key2.pem -passin pass:hello --RSAPublicKey_out -out public_key2.pem
```

#### Generating a Signature (Encrypted RSA)

```sh
openssl dgst -sha256 -out signature3.sig -sign private_key2.pem -passin pass:hello plain_text.txt
```

```sh
openssl pkeyutl -sign -in hash.sha256 -inkey private_key2.pem -passin pass:hello -out signature4.sig -pkeyopt digest:sha256
```

```sh
diff signature3.sig signature4.sig
```

#### Verifying the signature (Encrypted RSA)

Find out why OpenSSL dgst function accept a RSA Public key but only accepts a generic public key.#ToDo

Hence to avoid this problem lets convert our RSA Public Key to a generic SSL public public_key.pem

```sh
openssl rsa -RSAPublicKey_in -in public_key2.pem -out public_key3.pem -pubout
```

```sh
openssl dgst -sha256 -verify public_key3.pem -signature signature3.sig plain_text.txt
```

More inforation on the above transformation can be found at [How to transform between the two key styles](https://stackoverflow.com/a/29707204).

### Generating a private key (EC)

Similar to [Generating a private key (RSA)](#generating-a-private-key-rsa)

```sh
openssl genpkey -out private_key.pem -algorithm EC -pkeyopt ec_paramgen_curve:P-256 -pkeyopt ec_param_enc:named_curve
```
