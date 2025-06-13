# OpenSSL Encrypt & Decrypt

## Encrypt File

```bash
openssl enc -aes-256-cbc -salt -in plain.txt -out secret.enc
```

## Decrypt File

```bash
openssl enc -d -aes-256-cbc -in secret.enc -out plain.txt
```

## See Also

👉 [Overview](./openssl-overview.md)
