---
title: "Apache Web Server Mod Proxy"
related: true
categories:
  - linux
  - hacks
tags:
  - Apache
  - httpd
  - proxy
  - linux
toc: true  
---

## Introduction

This post is about the Apache (httpd) webserver's mod_proxy capability for my own reference.

I was required to setup an proxy a HTTP link to a web service running in another machine (RPI server). So, I remembered that I had did it my previous job as well trying to proxy http video service running in another video server.

## Flow Diagram

```shell
Client  <-----> Server  <-----> Remote
Machine         Machine         Machine
(HTTP request)  (Proxy to)     (Actual web service)
```

## Mod Proxy Setup

First setup the Apache httpd to enable mod_proxy and mod_ssl based on the linux distribution we use.

```shell
---------------  On Debian based systems ---------------
$ apache2ctl -t -D DUMP_MODULES   
OR
$ apache2ctl -M
---------------  On ArchLinux/RHEL based systems ---------------
$ apachectl -t -D DUMP_MODULES   
OR
$ httpd -M
$ apache2ctl -M
```

> In case of http to https redirect proxy, we need to ensure mod_ssl module is also loaded.

## Mod Proxy Configuration

After ensuring mod proxy is configured properly, we can write the configuration of mod_proxy.conf as follows,

```shell
<IfModule mod_proxy.c>

ProxyRequests Off

<Proxy *>
Order deny,allow
Allow from all
</Proxy>

SSLProxyEngine on

# This will redirect http://hostname/maps ==> googleapis.com/maps
ProxyPass /maps https://maps.googleapis.com/maps
ProxyPassReverse /maps https://maps.googleapis.com/maps

RewriteEngine  on

# This will redirect http://hostname/gateway1/controller/myarg/myvalue ==> http://10.0.0.1:8080/controller/myarg/myvalue
RewriteRule    "^/gateway1/controller/(.*)$"  "http://10.0.0.1:8080/controller/$1"  [P]
ProxyPass "/gateway1/" "http://10.0.0.1:8080/controller/$1" connectiontimeout=5 timeout=30
ProxyPassReverse "/gateway1/" "http://10.0.0.1:8080/controller/"

# HTTP ==> HTTPS proxying is also allowed.
# This will redirect http://hostname/gateway2/controller/myarg/myvalue ==> https://10.0.0.2:8080/controller/myarg/myvalue
RewriteRule    "^/gateway2/controller/(.*)$"  "https://10.0.0.2:8080/controller/$1"  [P]
ProxyPass "/gateway2/" "https://10.0.0.2:8080/controller/$1" connectiontimeout=5 timeout=30
ProxyPassReverse "/gateway2/" "https://10.0.0.2:8080/controller/"

</IfModule>
```

## Conclusion

I was able to redirect the same hostname request to different proxy servers namely gateway1 & gateway2 machines based on proper URL parsing, rewriting and proxying.
