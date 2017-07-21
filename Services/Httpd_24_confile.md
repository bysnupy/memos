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
