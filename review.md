## review

First of all, big thanks for this course to Anshul!

It's a great introduction course for System Administrators or Web Programmers who are just getting started with NGINX.

It's more a beginner course, not advanced. After this course you can safely start working with NGINX and dive deeper to more advanced features and write more advanced configs with the help of NGINX documentation or another resources.

To further explore NGINX and solidify its fundamental concepts I can recommend this section - http://nginx.org/en/books.html

You can get advanced book "The Complete NGINX Cookbook" by Derek DeJonghe here for free - https://www.nginx.com/resources/library/complete-nginx-cookbook/?utm_source=nginxorg&utm_medium=books&utm_campaign=ebooks

This is another free handbook "The NGINX Handbook" by Farhan Hasin Chowdhury here https://www.freecodecamp.org/news/the-nginx-handbook/

I would like to highlight a few cons:

- No certbot introduction. Certbot now is essential part of NGINX configuration, because you want to have all your traffic to the internet encrypted. Certbot is the best way to start using free Let's Encrypt SSL certs.
- You will not get best practices from this course, on the contrary, you will get the wrong idea about some concepts. For example, throughout the course you will use the main nginx.conf file. Which in fact is usually not touched at all. All user configs should be in /etc/nginx/conf.d or /etc/nginx/sites-avalaible for Ubuntu nginx package (it's an old deprecated way). I understand that it's ok for using the main config to demonstrate simple things clearly. But it would be much better to tell the students at some point in the course that they don't actually need to use the main config file in real world and that this approach is even the worst practice.
- Sometimes Anshul is wrong about some concepts or tools. There are also mistakes in Anshul's configs. For example, there are mistakes from "Avoiding the Top 10 NGINX Configuration Mistakes" list. It's not a big deal for experienced Sysadmins, but we are talking about beginner's course...
- The biggest con of the course is dead Q&A section. And as I understand, it's not the problem of a small user base. There are some questions without an answer for years. I started a thread about "29. Lab : Configure NGINX as Reverse Proxy". There are mistakes in Anshul's config presented in video. And what do you think happened? :) My question was just deleted.

Anyway, I'm glad that this course exists and I really appreciate Anshul's hard work, thanks and wish a nice day to all the students :)

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
