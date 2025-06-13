# OpenSSL Format Conversion

## PEM ↔ DER

```bash
# PEM to DER
openssl x509 -outform der -in mycert.pem -out mycert.der

# DER to PEM
openssl x509 -inform der -in mycert.der -out mycert.pem
```

## PEM + Key → PFX

```bash
openssl pkcs12 -export -inkey rsa_key.pem -in mycert.pem -out mycert.pfx
```

## See Also

👉 [Certificates](./openssl-certificates.md)  
👉 [Overview](./openssl-overview.md)
