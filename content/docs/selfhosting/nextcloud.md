---
title: NextCloud
type: docs
prev: docs/selfhosting/
---

### Nextcloud on Debian
Nextcloud is a flexible file synchronization and sharing solution.Nextcloud includes Nextcloud server( run on linux) and Nextcloud client.Nextcloud is a Free and Open Source community supported,with all enterprise features. 
In this doc lets try to install nextcloud server over Nginx and access it over browser client.

#### Nginx
Nginx is Free and Open Source web server which is now also used as reverse proxy,HTTP cache and load balancer.To setup nextcloud we can choose either nginx or apache as webserver.
Install and enable nginx service on the server

```bash
$ sudo apt install nginx -y
$ sudo systemctl start nginx
$ sudo systemctl enable nginx
```

### Installation Steps
#### Prerequisites for mannual installation.
https://docs.nextcloud.com/server/latest/admin_manual/installation/source_installation.html#prerequisites-for-manual-installation

##### Install php8.0 from deb.sury.org
Installing PHP from a third party repository https://deb.sury.org/ which contains the deb packaged version of the latest php and its modules.
This repository supports both Ubuntu and Debian.

```bash
if [ "$(whoami)" != "root" ]; then
    SUDO=sudo
fi

${SUDO} apt-get update
${SUDO} apt-get -y install lsb-release ca-certificates curl
${SUDO} curl -sSLo /usr/share/keyrings/deb.sury.org-php.gpg https://packages.sury.org/php/apt.gpg
${SUDO} sh -c 'echo "deb [signed-by=/usr/share/keyrings/deb.sury.org-php.gpg] https://packages.sury.org/php/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/php.list'
${SUDO} apt-get update
```

```bash
$ sudo apt policy php8.0
# check for any latest updated php package
```



```bash{filename="Install the packages mentioned"}
sudo apt install php8.0-xmlreader php8.0-curl php8.0-gd php8.0-mbstring php8.0-zip php8.0-fpm
```

Database connectors (either choose from MySQL/MariaDB and Postgresql)
```bash{filename="mysql database connector"}
$ sudo apt install mariadb-server php8.0-mysql
```

Caching
```bash{filename="php modules required for caching"}
$sudo apt install redis php8.0-redis
```

https://docs.nextcloud.com/server/latest/admin_manual/installation/nginx.html

```bash
vim /etc/php/8.0/cli/php.ini
update date.timezone = Asia/Kolkata
```

```
cd /var/www
sudo wget https://download.nextcloud.com/server/releases/latest.zip
sudo chown www-data:www-data /var/www/nextcloud -R
```

```sql
> create database nextcloud_db;
> create user nextcloud_user@localhost identified by 'deeproot';
> grant all privileges on nextcloud_db.* to nextcloud_user@localhost identified by 'deeproot';
> flush privileges
> exit
```

#### Nginx Configuration file.
paste the following in ```/etc/nginx/sites-enabled/nextcloud``` and remove any default files present.
and also make a symlink from ```/etc/nginx/sites-enabled/nextcloud``` to ```/etc/nginx/sites-available/nextcloud```

host ```nextcloud.vinay.com``` 
```
upstream php-handler {
    #server 127.0.0.1:9000;
    server unix:/var/run/php/php8.0-fpm.sock;
}

# Set the `immutable` cache control options only for assets with a cache busting `v` argument
map $arg_v $asset_immutable {
    "" "";
    default "immutable";
}

server {
    listen 80;
    server_name nextcloud.vinay.com; 

    # Path to the root of your installation
    root /var/www/nextcloud;

    # Use Mozilla's guidelines for SSL/TLS settings
    # https://mozilla.github.io/server-side-tls/ssl-config-generator/
    #ssl_certificate     /etc/ssl/nginx/cloud.example.com.crt;
    #ssl_certificate_key /etc/ssl/nginx/cloud.example.com.key;

    # Prevent nginx HTTP Server Detection
    server_tokens off;

    # HSTS settings
    # WARNING: Only add the preload option once you read about
    # the consequences in https://hstspreload.org/. This option
    # will add the domain to a hardcoded list that is shipped
    # in all major browsers and getting removed from this list
    # could take several months.
    #add_header Strict-Transport-Security "max-age=15768000; includeSubDomains; preload" always;

    # set max upload size and increase upload timeout:
    client_max_body_size 512M;
    client_body_timeout 300s;
    fastcgi_buffers 64 4K;

    # Enable gzip but do not remove ETag headers
    gzip on;
    gzip_vary on;
    gzip_comp_level 4;
    gzip_min_length 256;
    gzip_proxied expired no-cache no-store private no_last_modified no_etag auth;
    gzip_types application/atom+xml text/javascript application/javascript application/json application/ld+json application/manifest+json application/rss+xml application/vnd.geo+json application/vnd.ms-fontobject application/wasm application/x-font-ttf application/x-web-app-manifest+json application/xhtml+xml application/xml font/opentype image/bmp image/svg+xml image/x-icon text/cache-manifest text/css text/plain text/vcard text/vnd.rim.location.xloc text/vtt text/x-component text/x-cross-domain-policy;

    # Pagespeed is not supported by Nextcloud, so if your server is built
    # with the `ngx_pagespeed` module, uncomment this line to disable it.
    #pagespeed off;

    # The settings allows you to optimize the HTTP2 bandwitdth.
    # See https://blog.cloudflare.com/delivering-http-2-upload-speed-improvements/
    # for tunning hints
    client_body_buffer_size 512k;

    # HTTP response headers borrowed from Nextcloud `.htaccess`
    #add_header Referrer-Policy                   "no-referrer"       always;
    #add_header X-Content-Type-Options            "nosniff"           always;
    #add_header X-Download-Options                "noopen"            always;
    #add_header X-Frame-Options                   "SAMEORIGIN"        always;
    #add_header X-Permitted-Cross-Domain-Policies "none"              always;
    #add_header X-Robots-Tag                      "noindex, nofollow" always;
    #add_header X-XSS-Protection                  "1; mode=block"     always;

    # Remove X-Powered-By, which is an information leak
    fastcgi_hide_header X-Powered-By;

    # Add .mjs as a file extension for javascript
    # Either include it in the default mime.types list
    # or include you can include that list explicitly and add the file extension
    # only for Nextcloud like below:
    include mime.types;
    types {
        text/javascript js mjs;
    }

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
        # The rules in this block are an adaptation of the rules
        # in `.htaccess` that concern `/.well-known`.

        location = /.well-known/carddav { return 301 /remote.php/dav/; }
        location = /.well-known/caldav  { return 301 /remote.php/dav/; }

        location /.well-known/acme-challenge    { try_files $uri $uri/ =404; }
        location /.well-known/pki-validation    { try_files $uri $uri/ =404; }

        # Let Nextcloud's API for `/.well-known` URIs handle all other
        # requests by passing them to the front-end controller.
        return 301 /index.php$request_uri;
    }

    # Rules borrowed from `.htaccess` to hide certain paths from clients
    location ~ ^/(?:build|tests|config|lib|3rdparty|templates|data)(?:$|/)  { return 404; }
    location ~ ^/(?:\.|autotest|occ|issue|indie|db_|console)                { return 404; }

    # Ensure this block, which passes PHP files to the PHP process, is above the blocks
    # which handle static assets (as seen below). If this block is not declared first,
    # then Nginx will encounter an infinite rewriting loop when it prepends `/index.php`
    # to the URI, resulting in a HTTP 500 error response.
    location ~ \.php(?:$|/) {
        # Required for legacy support
        rewrite ^/(?!index|remote|public|cron|core\/ajax\/update|status|ocs\/v[12]|updater\/.+|oc[ms]-provider\/.+|.+\/richdocumentscode\/proxy) /index.php$request_uri;

        fastcgi_split_path_info ^(.+?\.php)(/.*)$;
        set $path_info $fastcgi_path_info;

        try_files $fastcgi_script_name =404;

        include fastcgi_params;
        fastcgi_pass unix:/var/run/php/php8.0-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $path_info;
        fastcgi_param HTTPS off;

        fastcgi_param modHeadersAvailable true;         # Avoid sending the security headers twice
        fastcgi_param front_controller_active true;     # Enable pretty urls
        #fastcgi_pass php-handler;
        fastcgi_intercept_errors on;
        fastcgi_request_buffering off;

        fastcgi_max_temp_file_size 0;
    }

    # Serve static files
    location ~ \.(?:css|js|mjs|svg|gif|png|jpg|ico|wasm|tflite|map)$ {
        try_files $uri /index.php$request_uri;
        add_header Cache-Control "public, max-age=15778463, $asset_immutable";
        access_log off;     # Optional: Don't log access to assets

        location ~ \.wasm$ {
            default_type application/wasm;
        }
    }

    location ~ \.woff2?$ {
        try_files $uri /index.php$request_uri;
        expires 7d;         # Cache-Control policy borrowed from `.htaccess`
        access_log off;     # Optional: Don't log access to assets
    }

    # Rule borrowed from `.htaccess`
    location /remote {
        return 301 /remote.php$request_uri;
    }

    location / {
        try_files $uri $uri/ /index.php$request_uri;
    }
}

```


#### OpenLDAP & NextCloud. 

install ldap php8.0 from deb sury repo
apt install php8.0-ldap
and then later enable LDAP user and group backend on nextcloud Apps

