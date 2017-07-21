## Httpd web server installation memo as compiling

### Environment

* httpd server version: 2.4.27
* dependencies (rpm or sources)

Package|Version|Type
-|-|-
apr|1.6.2|source
apr-util|1.6.0|source
pcre|8.41|source
openssl|1.0.2l|source
perl-devel|latest|rpm
expat-devel|latest|rpm

### Installing

#### Step0: Installing the dependencies

* perl-devel and expat-devel installation as yum
```bash
# yum install perl-devel expat-devel
```

* placing the apr and apr-util sources into the httpd sources
```bash
$ tar -jxvf apr-1.6.2.tar.bz2 -C httpd-2.4.27/srclib/
$ tar -jxvf apr-util-1.6.0.tar.bz2 -C httpd-2.4.27/srclib/
$ cd httpd-2.4.27/srclib/
$ mv apr-1.6.2 apr && mv apr-util-1.6.0 apr-util
```

* compiling the pcre source
```bash
$ configure --prefix=$HOME/path/to/pcre && make && make install
```

* compiling the openssl source

:star:You might add the installed openssl library path to global library path or LD_LIBRARY_PATH (or envvars).

```bash
$ config shared zlib threads --prefix=$HOME/path/to/ssl \
                             --openssldir=$HOME/path/to/ssl -fPIC && make && make install 
```

* mapping the openssl library path to shared library path

```bash
# cat > /etc/ld.so.conf.d/openssl-local-x86_64.conf <<EOF
/path/installed/openssl/homedir/lib
EOF

# ldconfig && ldconfig -p | grep /path/installed/openssl/homedir/lib
libssl.so.1.0.0 (libc6,x86-64) => /path/installed/openssl/homedir/lib/libssl.so.1.0.0
libssl.so (libc6,x86-64) => /path/installed/openssl/homedir/lib/libssl.so
libcrypto.so.1.0.0 (libc6,x86-64) => /path/installed/openssl/homedir/lib/libcrypto.so.1.0.0
libcrypto.so (libc6,x86-64) => /path/installed/openssl/homedir/lib/libcrypto.s
#
```

### Step1: Installing httpd web server

```bash
$ configure  --prefix=$HOME/path/to/httpd \
             --with-included-apr --with-pcre=$HOME/path/to/pcre --enable-mpms-shared=all \
             --enable-mods-shared=most --with-ssl=$HOME/path/to/ssl && make && make install
```

### Step2: Verifying the installation

```bash
$ cd $HOME/path/to/httpd/bin
$ ./httpd -V
Server version: Apache/2.4.27 (Unix)
Server built:   Jul 21 2017 16:03:39
Server's Module Magic Number: 20120211:68
Server loaded:  APR 1.6.2, APR-UTIL 1.6.0
Compiled using: APR 1.6.2, APR-UTIL 1.6.0
Architecture:   64-bit
Server MPM:     event
  threaded:     yes (fixed thread count)
    forked:     yes (variable process count)
Server compiled with....
 -D APR_HAS_SENDFILE
 -D APR_HAS_MMAP
 -D APR_HAVE_IPV6 (IPv4-mapped addresses enabled)
 -D APR_USE_SYSVSEM_SERIALIZE
 -D APR_USE_PTHREAD_SERIALIZE
 -D SINGLE_LISTEN_UNSERIALIZED_ACCEPT
 -D APR_HAS_OTHER_CHILD
 -D AP_HAVE_RELIABLE_PIPED_LOGS
 -D DYNAMIC_MODULE_LIMIT=256
...snip...
```

Done.
