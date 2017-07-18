## How to use CIDR format at the access control based on mod_rewrite on the httpd service

### Introduction
We can use CIDR format to REMOTE_ADDR or REMOTE_HOST at the "RewriteCond" directive if you're using httpd 2.4 or later.

Related references: [Expressions in Apache httpd server](http://httpd.apache.org/docs/2.4/expr.html)

### Enviorment

Item|Value
-|-
  httpd| Apache/2.4.18
  Required modules| core_module(with PCRE library) , rewrite_module, proxy_module, proxy_http_module
  
### Configuration Summary
When you limit a large number of source IPs with complicated conditions on the Apache Web Server 2.2 or former version, we had no choice but to use Regular Expressions in the "RewriteCond" directives.
But as of 2.4 version we can use a new expression parser allows to specify complex conditions including a CIDR format. 
The test environment assumes following access flow, It has http and https virtual hosts processing requests respectively and existing SORRY server just displaying SORRY page with 503 status code.

![process flow figure1](https://github.com/bysnupy/memos/blob/master/Services/images/httpd__process_flow_figure1.png)

It just can be forwarded all requests to SORRY page by Web server except the specific IPs.

### Configuration Steps
We assume that the configurations are completed properly about Apache and Tomcat stack, and the exception IP range is 192.168.123.0/24 here.

the configuration file tree is as follows:

```bash
|-- [conf/httpd.conf: Main configuration file]
      |--extra
          |--[vhost-http.conf: Http virtual host]
          |--[vhost-https.conf: Https virtual host]
          |--[vhost-common.conf: Http and Https common configurations are included from this file]
          |--[vhost-rewrite.conf: Main Rewrite configuration file]
```

#### Step1: Configure the Rewrite rules in accordance with specifics

It's simple to configure CIDR rules to the "RewriteCond", as follows: be careful to the nested quotations marks and exclamation mark.

```httpd
RewriteCond expr "! -R '192.168.123.0/24'"
```

It means that it's false if the IP range is "192.168.123.0/24". 

```httpd
RewriteCond expr "-R '192.168.123.0/24'"
```

It means that it's true if the IP range is "192.168.123.0/24".

It is the full configuration of the "vhost-rewrite.conf".

```bash
  $ vim conf/extra/vhost-rewrite.conf
<ifmodule rewrite_module>
  RewriteEngine On
  #- DEBUG Logging
  #LogLevel alert rewrite:trace3
  
  #- Conditional rules
  RewriteCond       expr            "! -R '192.168.123.0/24'"
  RewriteRule       ^/(.*)$         http://sorry.server.ip_or_host/$1 [P]
  ProxyPasssReverse /               http://sorry.server.ip_or_host/
</ifmodule> 
```

#### Step2: Including the Rewrite configuration file into Http and Https virtual host section respectively

"vhost-common.conf" is just including "vhost-rewrite.conf" file and just be included "vhost-http.conf" and "vhost-https.conf" respectively.
The configuration file including tree is as follows.
The other solutions are existing, it's using RewriteOptions directives.
But It is not intuitive.

```bash
|--[httpd.conf]
    |
    |--[vhost-http.conf]
    |   ^
    |   |
    |  (include)
    |   |
    |   |<---[vhost-common.conf]<--(include)--[vhost-rewrite.conf]
    |   |
    |  (include)
    |   |
    |   v
    |--[vhost-https.conf]
```

"vhost-common.conf" file is included into "VirtualHost" block directive to avoid priority problem between proxy rules, such as follows:

```httpd
<VirtualHost *:80>
  ...snip...
  Include conf/extra/c21-common.conf
</VirtualHost> 
```

Done.
