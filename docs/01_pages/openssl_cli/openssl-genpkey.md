# OpenSSL `genpkey`

`genpkey` is the recommended modern command to generate private keys.

## Examples

**Generate RSA key:**

```bash
openssl genpkey -algorithm RSA -out rsa_key.pem -pkeyopt rsa_keygen_bits:2048
```

**Generate EC key:**

```bash
openssl genpkey -algorithm EC -out ec_key.pem -pkeyopt ec_paramgen_curve:P-256
```

**Generate DH key (requires params):**

```bash
openssl genpkey -paramfile dhparam.pem -out dhkey.pem
```

## See Also

👉 [Generate Parameters](./openssl-genparam.md)  
👉 [Overview](./openssl-overview.md)
