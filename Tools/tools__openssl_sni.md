# OpenSSL for SNI check

## `--servername` is required to check as SNI
~~~
echo | openssl s_client -connect test.example.com:443 -servername test.example.com | openssl x509 -text
~~~
