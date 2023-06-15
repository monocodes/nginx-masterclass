## reverse-proxy-discussion
Hi everyone!

I'm sorry, but most part of **29. Lab : Configure NGINX as Reverse Proxy** Lecture is a complete mess. **You can't use 2 virtual hosts with the same port that way.**

Also, I don't understand why the lecturer always uses main **nginx.conf** config. It's simple thing that main config may be edited only by package maintainers. All users as a best practice have to use */etc/nginx/sites-enabled* and */etc/nginx/sites-available* for their configs. With that approach you can always safely update and reinstall nginx package from ubuntu repos.

Here are great reference on [Nginx Web Server / Directory Structure](https://wiki.debian.org/Nginx/DirectoryStructure)

I discovered 3 ways to achieve what the lecturer trying to do. There are maybe more.

*Configs for Reverse Proxy server.*

**1. nginx-reverse-proxy-v1-php.conf**

Requests on ALL content proxied to static server.

Requests on **ALL** **php content** proxied to php app server via `location ~ \.php {...`

Access the php app via proxy like this - <http://your-reverse-proxy-server/index.php>, in my case it is <http://ub22-nginx-rp/index.php>

```nginx
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
        worker_connections 768;
        # multi_accept on;
}

http {

    sendfile on;
    tcp_nopush on;
    types_hash_max_size 2048;

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;

    server {
        listen 80;
        server_name ub22-nginx-rp;

        location / {
            proxy_pass http://ub22-nginx;
        }

        location ~ \.php {
            proxy_pass http://ub22-nginx-app;
        }
    }
}
```

**2. nginx-reverse-proxy-v2-path.conf**

Requests on ALL content proxied to static server.

Requests on **ALL content after** `**/php**` proxied on php app server via `location /php {...`

The main difference here that on php app server you need to have `/usr/share/nginx/html/**php**/index.php` because it is how `location` directive work. More info here - [NGINX Reverse Proxy](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/).

Access the php app via proxy like this - **<http://your-reverse-proxy-server/php>**

In my case it is **<http://ub22-nginx-rp/php>**

```nginx
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
        worker_connections 768;
        # multi_accept on;
}

http {

    sendfile on;
    tcp_nopush on;
    types_hash_max_size 2048;

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;

    server {
        listen 80;
        server_name ub22-nginx-rp;

        location / {
            proxy_pass http://ub22-nginx;
        }

        location /php {
            proxy_pass http://ub22-nginx-app;
        }
    }
}
```

**3. nginx-reverse-proxy-v3-port.conf**

Requests on ALL content proxied to static server.

Requests on **ALL** **content with port 8080** proxied to php app server via `listen 8080;...`

Access the php app via proxy like this - **<http://your-reverse-proxy-server:8080>**

In my case it is **<http://ub22-nginx-rp:8080>**

```nginx
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
        worker_connections 768;
        # multi_accept on;
}

http {

    sendfile on;
    tcp_nopush on;
    types_hash_max_size 2048;

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;

    server {
        listen 80;
        server_name ub22-nginx-rp;

        location / {
            proxy_pass http://ub22-nginx;
        }

    }

    server {
        listen 8080;
        server_name ub22-nginx-rp;

        location / {
            proxy_pass http://ub22-nginx-app;
        }
    }
}
```

1. some text

   ```nginx
   code block
   ```

2. another text

   ```nginx
   code block 2
   ```

---

## 42. Prevent DOS attack or Limit the Service

test url with `siege`

- `-r` - --reps=NUM, REPS, number of times to run the test.

```sh
siege -v -r 2000 -c 500 https://ub22-nginx/assets/js/custom.js
```

`-r` is number of times to run the test. Not number of requests.

`-c` - --concurrent=NUM, CONCURRENT users, default is 10. And it is limited to 255 if not changed in ~/.siege/siege.config
