## Miscellaneous commands memo

* Verifying the file's signature

```bash
gpg --verify gnupg-2.0.30.tar.bz2.sig gnupg-2.0.30.tar.bz2
```

* Extermine your external IP

```bash
-- 1
dig +short myip.opendns.com @resolver1.opendns.com

-- 2
curl -s http://whatismyip.akamai.com/

-- 3
curl -s https://4.ifcfg.me/
```
