# Table of Contents

- [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
  - [Asymmetric Key Cryptography](#asymmetric-key-cryptography)
    - [Generating a private key (RSA)](#generating-a-private-key-rsa)
    - [Extracting the public Key (RSA)](#extracting-the-public-key-rsa)
    - [Generating a signature (RSA)](#generating-a-signature-rsa)
    - [Verifying a signature (RSA)](#verifying-a-signature-rsa)
    - [Encrypting a file (RSA)](#encrypting-a-file-rsa)
    - [Decrypting a file (RSA)](#decrypting-a-file-rsa)


## Introduction

OpenSSL is a general purpose cryptography library / utility.
In this article we will be exploring the Command Line Interface / Utility part of OpenSSL.
Before we get started please make a note of the following points:

- OpenSSL assumes the keys in the arguments to be of the PEM(privacy Enhanced Mail) format if not specified.
- OpenSSL considers the keys in arguments to be private keys by default,

## Asymmetric Key Cryptography

The various asymmetric key cryptography algorithms supported are RSA, RSA-PSS, EC, X25519, X448, ED25519 and ED448.
GenPKey is a utility to generate private keys in OpenSSL.

### Generating a private key (RSA)

The sub-util used for generating a private key is ```genpkey```.

- The default size of RSA key is 2048 bits.
- The default number of prime numbers used is 2.
- The default public exponent is 65537.
- The RSA key sizes is a minimum of 1024 bits.

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

The public key can be extracted from the private key using the ```rsa``` sub-util.

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

Signature can be created either using the ```dgst``` sub util or using the ```pkeyutl``` sub util.

Lets create a plain text:

```sh
echo "Wonderful Wolrd!" > plain_text.txt
```

There are two ways we can sign a plain text as described above.

Let us first generate the signature using the ```dgst``` utl.

```sh
openssl dgst -sha256 -out signature.sig -sign private_key.pem plain_text.txt
```

Let us create another signature using the second method.
For this first lets generate the sha2-256 digest of the plain_text.txt.

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

### Verifying a signature (RSA)

Prerequisites
[Generating a signature (RSA)](#generating-a-signature-rsa) and [Extracting the public Key (RSA)](#extracting-the-public-key-rsa)

The verify signature is similar to [Generating a signature (RSA)](#generating-a-signature-rsa)

```sh
openssl dgst -sha256 -verify public_key.pem -signature signature.sig plain_text.txt
```

The second method is the roundabout way of calculating hash and then verifying the signature.

But this was not successful for me - hence please update when you get the time.

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
