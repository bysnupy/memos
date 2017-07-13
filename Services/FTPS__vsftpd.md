## FTPS(FTP over SSL/TLS) configuration with vsftpd

### Introduction
The implementation of the FTPS service with vsftpd(official site), when I had simulated the backend tasks.

### Environment
The vsftpd package was installed by yum

Item|Value
-|-
Hostname| repov1.host.local
OS| CentOS 7.2
Packagename| vsftpd
Version| 3.0.2-11.el7_2

### Configuration steps

#### Step1: Issuing a self-signed certificate with OpenSSL

I will create the certificate that signed with own root CA.

##### 1. Issuing the self-signed certificate for Certificate Authority

```bash
# openssl genrsa -out /etc/pki/CA/private/root-cakey.pem 2048

# openssl req -x509 -nodes -new -days 3650 -sha256 -key /etc/pki/CA/private/root-cakey.pem \
          -out /etc/pki/CA/root-cacert.pem
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:JP
State or Province Name (full name) []:Tokyo
Locality Name (eg, city) [Default City]:Minato
Organization Name (eg, company) [Default Company Ltd]:Local Company
Organizational Unit Name (eg, section) []:Infrastructure Division
Common Name (eg, your name or your server's hostname) []:
Local Company Root CA
Email Address []:
```

##### 2. Issuing the self-signed certificate for vsftpd

```bash
# openssl genrsa /etc/pki/tls/private/repov1.host.local-key.pem 2048

# openssl req -new -nodes -sha256 -out /etc/pki/tls/repov1.host.local.csr \
          -key /etc/pki/tls/private/repov1.host.local-key.pem

You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:JP
State or Province Name (full name) []:Tokyo
Locality Name (eg, city) [Default City]:Minato
Organization Name (eg, company) [Default Company Ltd]:Local Company
Organizational Unit Name (eg, section) []:Infrastructure Division
Common Name (eg, your name or your server''s hostname) []:repov1.host.local
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
# openssl x509 -sha256 -days 3650 -req -in /etc/pki/tls/repov1.host.local.csr \
          -CAcreateserial -CA /etc/pki/CA/root-cacert.pem -CAkey /etc/pki/CA/private/root-cakey.pem \
          -out /etc/pki/tls/certs/repov1.host.local-cert.pem
```

##### 3. Deployment of the self-signed certificate to FTPS client servers

I had copied the certificate to /etc/pki/tls/certs/repov1.host.local-cert.pem on the FTPS client servers

#### Step2: Configuration for vsftpd.conf

The following is a snippet of FTPS configurations, not all configurations. (config options manual)

```bash
# vim /etc/vsftpd/vsftpd.conf
#> FTPS configuration
#+ enable the SSL/TLS
ssl_enable=YES

#+ the protocols whether enabled or not
ssl_tlsv1=YES

#+ allow and force anonymous, local users for using SSL/TLS
#- for anonymous users
allow_anon_ssl=YES
force_anon_data_ssl=YES
force_anon_logins_ssl=YES
#- for local users
force_local_data_ssl=YES
force_local_logins_ssl=YES

#+ specify the certificate and private key files
rsa_cert_file=/etc/pki/tls/certs/repov1.host.local-cert.pem
rsa_private_key_file=/etc/pki/tls/private/repov1.host.local-key.pem

#+ change the control and data port, 990/TCP for control channel, 989/TCP for data channel
listen_port=990
ftp_data_port=989
#+ if you started daemon with the passive mode, you would need to limit the passive ports.
pasv_min_port=60000
pasv_max_port=60010
```

#### Step3: Test the FTPS service

I used lftp client here to verify the FTPS configurations, but WinSCP can be used here too.
And FTPS requires two connections per session like FTP: a control channel and a data channel,
you need to allow the ports before test if firewalld or iptables is activated.

```bash
$ lftp -e 'set ftp:ssl-force true;set ssl:ca-file "/etc/pki/tls/certs/repov1.host.local-cert.pem"' \
       -u anonymous repov1.host.local
Password:
lftp anonymous@repov1.host.local:~> ls
---- Connecting to repov1.host.local (192.168.122.84) port 990
<--- 220 (vsFTPd 3.0.2)
---> FEAT
<--- 211-Features:
<---  AUTH TLS

... snip ...

---> AUTH TLS
<--- 234 Proceed with negotiation.
---> OPTS UTF8 ON
Certificate: C=JP,ST=Tokyo,L=Minato,O=Local Company,OU=Infrastructure Division,CN=repov1.host.local
 Issued by: C=JP,ST=Tokyo,L=Minato,O=Local Company,OU=Infrastructure Division,CN=Local Company Root CA
  Trusted

...snip...
```

Test with WinSCP as follows.
![lsyncd oneway sync img1](https://github.com/bysnupy/memos/blob/master/Services/images/ftps__winscp1.png)

Done.
