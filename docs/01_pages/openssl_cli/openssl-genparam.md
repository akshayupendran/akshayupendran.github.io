# OpenSSL `genparam`

`genparam` generates parameters needed by some algorithms (DH, DSA).

## Example: Generate DH Params

```bash
openssl genparam -algorithm DH -out dhparam.pem -pkeyopt dh_paramgen_prime_len:2048
```

Then generate a DH private key:

```bash
openssl genpkey -paramfile dhparam.pem -out dhkey.pem
```

## See Also

👉 [Generate Private Keys](./openssl-genpkey.md)  
👉 [Overview](./openssl-overview.md)
