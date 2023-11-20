---
title: Wordpress behind nginx reverse proxy
date: 2016-06-22 17:52:50.000000000 +03:00
type: post
categories:
- How to
tags:
- Linux
- NGINX
- Wordpress
permalink: "/wordpress-behind-nginx-reverse-proxy/"
---
I had the task to get a wordpress website working on the internet. The website was only used locally under the domain intranet.local. So the customer wanted to publish it to the internet under the domain example.com but also under the domain intranet.local.

So the only way was to use a reverse proxy. I did it with the following configuration on nginx :

```
server {
 listen 80;
 server_name www.example.com;

location / {
 proxy_pass http://www.intranet.local;
 sub_filter_once off;
 sub_filter 'www.intranet.local' 'www.example.com';
 sub_filter_types *;
 }
 }
```

The above configuration also rewrites answers going to the user's browser.

Also I edited wp-config.php and added the following after $table\_prefix

```
/*
 * Handle reverse proxy
 */
define('WP_SITEURL', 'http://www.intranet.local');
define('WP_HOME', 'http://www.intranet.local');
```
