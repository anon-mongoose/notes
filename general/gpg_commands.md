
## To list existing public keys:
```bash
gpg -k
```


## To list existing private keys:
```bash
gpg -K
```


## To generate a key:
```
gpg --full-generate-key --expert
```

Prefer either RSA 4096 or Curve 25519.


## To edit a key:
```bash
gpg --edit-key --expert <key id>
```


## To delete private keys and sub-keys:
```bash
gpg --delete-secret-key <key id>
```


## To delete both public and private keys:
```bash
gpg --delete-secret-and-public-key <key id>
```


## To list existing keys:
```bash
gpg --list-signatures
```


## To checks signatures of existing keys:
```bash
gpg --check-signatures
```


## To sign a key with another one:
```bash
gpg -u <key id that will sign> --sign-key <key id that will be signed>
```


## To export a public key:
```bash
gpg --armor --output pubkey.gpg --export <key id>
```


## To export a private key:
```bash
gpg --armor --output privkey.gpg --export-secret-keys
```


## To expert private sub-keys only:
```bash
gpg --armor --output private-subkeys.gpg --export-secret-subkeys <key id>
```


## To generate a revokation certificate:
```bash
gpg --output revoke.asc --gen-revoke <key id>
```


## To check a smartcard's status:
```bash
gpg --card-status
```

If an error occured when running this command, restart the `pcscd` daemon and try again:
```bash
systemctl restart pcscd
```


## To add existing sub-keys to a smartcard:
```bash
gpg --edit-key --expert <key id>
gpg> list
gpg> key 1 # the first ssb in the list, a '*' should appear next to it
gpg> keytocard
```

