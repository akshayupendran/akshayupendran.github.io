# OpenSSL Cheatsheet

✅ Check version:
```bash
openssl version -a
```

✅ Generate RSA key:
```bash
openssl genpkey -algorithm RSA -out key.pem -pkeyopt rsa_keygen_bits:2048
```

✅ Create CSR:
```bash
openssl req -new -key key.pem -out csr.csr
```

✅ Self-sign cert:
```bash
openssl req -x509 -key key.pem -out cert.pem -days 365
```

✅ Encrypt/Decrypt:
```bash
openssl enc -aes-256-cbc -salt -in plain.txt -out secret.enc
openssl enc -d -aes-256-cbc -in secret.enc -out plain.txt
```

✅ Check cert:
```bash
openssl x509 -in cert.pem -text -noout
```

✅ Test server:
```bash
openssl s_client -connect example.com:443
```

## See Also

👉 [Overview](./openssl-overview.md)
