---
title: Nextcloud on OMV 5
date: 2020-04-27 11:43:24.000000000 +03:00
type: post
categories:
- How to
tags:
- Debian
- Linux
- Nextcloud
- OMV 5
permalink: "/nextcloud-on-omv-5/"
---
As the title states. This solution is without the use of docker. It is just a straight forward installation of Nextcloud 18.0.3 on OMV 5 using Nginx (php-fpm)  with PHP 7.3 that is already installed and MariaDB as our database server. In order to work on port 80 or port 443, we will have to change the OMV web ports to 8080 and 8443. You can achieve this via "General settings" on the OMV web interface. This guide asumes that Nextcloud will user ports TCP/80 and TCP/443. Everything that is marked RED, needs your attention.

Let's start with some packages.

```
apt install mariadb-server php-xml php-cli php-cgi php-mysql php-mbstring php-gd php-curl php-zip wget unzip php-imagick php-intl php-gmp -y
```

Download nextcloud and place it on folder /var/www

```
cd /usr/src
wget https://download.nextcloud.com/server/releases/nextcloud-18.0.4.zip
unzip nextcloud-18.0.4.zip
mv nextcloud /var/www/
chown -R www-data:www-data /var/www/nextcloud/
chmod -R 755 /var/www/nextcloud/
```

Now, we have to create a new pool on the fpm, in order to adjust php settings for nextcloud and not mess up the OMV web part.

```
nano /etc/php/7.3/fpm/pool.d/nextcloud.conf

[nextcloud]
user = www-data
group = www-data

listen = /run/php/php7.3-fpm-nextcloud.sock
listen.owner = www-data
listen.group = www-data
listen.mode = 0600

pm = dynamic
pm.max_children = 120
pm.start_servers = 12
pm.min_spare_servers = 6
pm.max_spare_servers = 18
chdir = /

php_value[include_path] = ".:/usr/share/php:/var/www/nextcloud"
php_value[upload_max_filesize] = 512M
php_value[post_max_size] = 512M
php_value[max_execution_time] = 300
php_value[date.timezone] = Europe/Athens
php_value[expose_php] = Off
php_value[memory_limit] = 512M

env[PATH] = /usr/local/bin:/usr/bin:/bin
```

To improve performance we have to enable opcache and use redis

```
nano /etc/php/7.3/fpm/conf.d/10-opcache.ini

configuration for php opcache module
; priority=10
zend_extension=opcache.so

opcache.enable=1
opcache.enable_cli=1
opcache.interned_strings_buffer=8
opcache.max_accelerated_files=10000
opcache.memory_consumption=128
opcache.save_comments=1
opcache.revalidate_freq=1
```

Install redis and make changes to config.php.

```
apt-get install redis-server php-redis
systemctl start redis-server
systemctl enable redis-server
```

```
nano /var/www/nextcloud/config/config.php

 'memcache.local' => '\\OC\\Memcache\\Redis',
'memcache.locking' => '\\OC\\Memcache\\Redis',
'redis' =>
array(
'host' => 'localhost',
'port' => '6379',
),
```

Set up our database.

```
systemctl enable mariadb

mysql_secure_installation

mysql -u root -p

CREATE DATABASE nextclouddb;
CREATE USER 'nextclouduser'@'localhost' IDENTIFIED BY 'changeme';
GRANT ALL ON nextclouddb.* TO 'nextclouduser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

Then we have to create a server block from Nginx. This block has SSL enabled so prior to that, you should have SSL certificates created. I point out in bold where changes must be made and also where the SSL certificates should be declared.

```
nano /etc/nginx/sites-available/nextcloud

server {
    listen [::]:80;
    server_name myserver.mydomain.com;

    # Enforce HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen [::]:443;
    server_name myserver.mydomain.com;

    # Use Mozilla's guidelines for SSL/TLS settings
    # https://mozilla.github.io/server-side-tls/ssl-config-generator/
    ssl_certificate     /etc/ssl/certs/nextcloud.crt;
    ssl_certificate_key /etc/ssl/private/nextcloud.key;

    # HSTS settings
    # WARNING: Only add the preload option once you read about
    # the consequences in https://hstspreload.org/. This option
    # will add the domain to a hardcoded list that is shipped
    # in all major browsers and getting removed from this list
    # could take several months.
    #add_header Strict-Transport-Security "max-age=15768000; includeSubDomains; preload;" always;

    # set max upload size
    client_max_body_size 512M;
    fastcgi_buffers 64 4K;

    # Enable gzip but do not remove ETag headers
    gzip on;
    gzip_vary on;
    gzip_comp_level 4;
    gzip_min_length 256;
    gzip_proxied expired no-cache no-store private no_last_modified no_etag auth;
    gzip_types application/atom+xml application/javascript application/json application/ld+json application/manifest+json application/rss+xml application/vnd.geo+json application/vnd.ms-fontobject application/x-font-ttf application/x-web-app-manifest+json application/xhtml+xml application/xml font/opentype image/bmp image/svg+xml image/x-icon text/cache-manifest text/css text/plain text/vcard text/vnd.rim.location.xloc text/vtt text/x-component text/x-cross-domain-policy;

    # Pagespeed is not supported by Nextcloud, so if your server is built
    # with the `ngx_pagespeed` module, uncomment this line to disable it.
    #pagespeed off;

    # HTTP response headers borrowed from Nextcloud `.htaccess`
    add_header Referrer-Policy                      "no-referrer"   always;
    add_header X-Content-Type-Options               "nosniff"       always;
    add_header X-Download-Options                   "noopen"        always;
    add_header X-Frame-Options                      "SAMEORIGIN"    always;
    add_header X-Permitted-Cross-Domain-Policies    "none"          always;
    add_header X-Robots-Tag                         "none"          always;
    add_header X-XSS-Protection                     "1; mode=block" always;

    # Remove X-Powered-By, which is an information leak
    fastcgi_hide_header X-Powered-By;

    # Path to the root of your installation
    root /var/www/nextcloud;

    # Specify how to handle directories -- specifying `/index.php$request_uri`
    # here as the fallback means that Nginx always exhibits the desired behaviour
    # when a client requests a path that corresponds to a directory that exists
    # on the server. In particular, if that directory contains an index.php file,
    # that file is correctly served; if it doesn't, then the request is passed to
    # the front-end controller. This consistent behaviour means that we don't need
    # to specify custom rules for certain paths (e.g. images and other assets,
    # `/updater`, `/ocm-provider`, `/ocs-provider`), and thus
    # `try_files $uri $uri/ /index.php$request_uri`
    # always provides the desired behaviour.
    index index.php index.html /index.php$request_uri;

    # Default Cache-Control policy
    expires 1m;

    # Rule borrowed from `.htaccess` to handle Microsoft DAV clients
    location = / {
        if ( $http_user_agent ~ ^DavClnt ) {
            return 302 /remote.php/webdav/$is_args$args;
        }
    }

    location = /robots.txt {
        allow all;
        log_not_found off;
        access_log off;
    }

    # Make a regex exception for `/.well-known` so that clients can still
    # access it despite the existence of the regex rule
    # `location ~ /(\.|autotest|...)` which would otherwise handle requests
    # for `/.well-known`.
    location ^~ /.well-known {
        # The following 6 rules are borrowed from `.htaccess`

        rewrite ^/\.well-known/host-meta\.json  /public.php?service=host-meta-json  last;
        rewrite ^/\.well-known/host-meta        /public.php?service=host-meta       last;
        rewrite ^/\.well-known/webfinger        /public.php?service=webfinger       last;
        rewrite ^/\.well-known/nodeinfo         /public.php?service=nodeinfo        last;

        location = /.well-known/carddav     { return 301 /remote.php/dav/; }
        location = /.well-known/caldav      { return 301 /remote.php/dav/; }

        try_files $uri $uri/ =404;
    }

    # Rules borrowed from `.htaccess` to hide certain paths from clients
    location ~ ^/(?:build|tests|config|lib|3rdparty|templates|data)(?:$|/)  { return 404; }
    location ~ ^/(?:\.|autotest|occ|issue|indie|db_|console)              { return 404; }

    # Ensure this block, which passes PHP files to the PHP process, is above the blocks
    # which handle static assets (as seen below). If this block is not declared first,
    # then Nginx will encounter an infinite rewriting loop when it prepends `/index.php`
    # to the URI, resulting in a HTTP 500 error response.
    location ~ \.php(?:$|/) {
        fastcgi_split_path_info ^(.+?\.php)(/.*)$;
        set $path_info $fastcgi_path_info;

        try_files $fastcgi_script_name =404;

        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $path_info;
        fastcgi_param HTTPS on;

        fastcgi_param modHeadersAvailable true;         # Avoid sending the security headers twice
        fastcgi_param front_controller_active true;     # Enable pretty urls
        fastcgi_pass unix:/run/php/php7.3-fpm-nextcloud.sock;

        fastcgi_intercept_errors on;
        fastcgi_request_buffering off;
    }

    location ~ \.(?:css|js|svg|gif)$ {
        try_files $uri /index.php$request_uri;
        expires 6M;         # Cache-Control policy borrowed from `.htaccess`
        access_log off;     # Optional: Don't log access to assets
    }

    location ~ \.woff2?$ {
        try_files $uri /index.php$request_uri;
        expires 7d;         # Cache-Control policy borrowed from `.htaccess`
        access_log off;     # Optional: Don't log access to assets
    }

    location / {
        try_files $uri $uri/ /index.php$request_uri;
    }
}

```

Let's enable the above block.

```
cd /etc/nginx/sites-enabled
ln -s ../sites-available/nextcloud nextcloud
```

For the Nextcloud storage location, I used a folder on my RAID storage. The installation of Nextcloud, asks for data storage path. By default, it places data files inside /var/www/nextcloud. You can skip this step, if you don't want to change the default storage path.

```
mkdir /srv/dev-disk-by-id-md-name/cloud
chown www-data:www-data /srv/dev-disk-by-id-md-name/cloud
```

Restart some services

```
systemctl restart nginx
systemctl restart php7.3-fpm
```

Create cron backgroud job

```
crontab -e -u www-data
```

Add the following line

```
*/15 * * * * php -f /var/www/nextcloud/cron.php
```

Begin the installation on **https://yourserveripaddress**

On **Administration > Basic settings**, set background jobs to cron.
