# OpenSSL Certificate Signing Request (CSR)

Use CSR to request a signed certificate.

## Create CSR

```bash
openssl req -new -key rsa_key.pem -out mycsr.csr
```

Skip prompts:

```bash
openssl req -new -key rsa_key.pem -out mycsr.csr \
  -subj "/C=IN/ST=TN/L=Chennai/O=Example Ltd/OU=IT/CN=example.com"
```

## See Also

👉 [Private Keys](./openssl-genpkey.md)  
👉 [Certificates](./openssl-certificates.md)  
👉 [Overview](./openssl-overview.md)
