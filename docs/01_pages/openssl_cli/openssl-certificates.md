# OpenSSL Certificates

## Self-signed Certificate

```bash
openssl req -x509 -key rsa_key.pem -in mycsr.csr -out mycert.pem -days 365
```

Or directly:

```bash
openssl req -x509 -newkey rsa:2048 -keyout rsa_key.pem -out mycert.pem -days 365
```

## View Certificate Details

```bash
openssl x509 -in mycert.pem -text -noout
```

## Verify Key and Certificate

```bash
openssl x509 -noout -modulus -in mycert.pem | openssl md5
openssl rsa -noout -modulus -in rsa_key.pem | openssl md5
```

## See Also

👉 [CSR](./openssl-csr.md)  
👉 [Overview](./openssl-overview.md)
