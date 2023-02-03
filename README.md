# Nginx- getting started

## update and upgrade system

run
`$ sudo apt update && sudo apt upgrade`

then install nginx by running
`$ sudo apt install nginx`

To confirm the installation, run

```bash
$ nginx -h
# shows help menu

$ nginx -v
nginx version: nginx/1.18.0 (Ubuntu)
```

---

## run the build script in react app

```bash
# cd into root
$ cd tsc-p-react

$ npm run build
> tsc-p-react@0.0.0 build
> tsc && vite build

vite v4.0.2 building for production...
✓ 383 modules transformed.
dist/index.html                     0.46 kB
dist/assets/index-f81f53a3.css    194.11 kB │ gzip:    27.22 kB
dist/assets/index-707af00b.js   4,131.63 kB │ gzip: 1,262.92 kB
```

_this generates dist dir in project root_

---

## create configuration file in sites-available

> file is named tsc-p-r.conf in this case

Open configuration file in editor

```bash
$ sudo nano /etc/nginx/sites-available/tsc-p-r.conf
```

and, copy the code below:

```conf
server {
  listen 80;
  server_name 192.168.2.102;
  root /home/hrutvik_/tsc-p-react/dist;
  index index.html;
  location / {
  try_files $uri /index.html =404;
  }
}
```

or

```conf
server {
  listen 80;
  listen [::]:80;
  server_name 172.30.16.1;
  root /home/hrutvik_/tsc-p-react/dist;
  index index.html;
  location / {
  try_files $uri /index.html =404;
  }
}
```

save and exit

---

## create symlink-

Link configuration file created in sites-availaible dir
to sites-enabled dir
> this dir is read by nginx/nginx.conf

```bash
$ sudo ln -s /etc/nginx/sites-available/tsc-p-r.conf /etc/nginx/sites-enabled/
$
```

> Confirm link is working by reading the conf file in nginx/sites-enabled -"as this dir is read by nginx/nginx.conf
>
>  ```bash
>  $ cat /etc/nginx/sites-enabled/tsc-p-r.conf
>  # if this outputs the file you edited, proceed..
>  ```

---

## Test the configuration file to make sure everything is good

```bash
$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

> If the output is OK, proceed

---

## To restart nginx

run

```bash
$ sudo systemctl restart nginx
```

## This does not work in WSL2, as _WSL2 uses **init** instead of **systemd**_

[reference](https://linuxhandbook.com/system-has-not-been-booted-with-systemd/)

run `ps -p 1 -o comm=` which outputs the value() of PID 1 variable(the first process that runs on your system),\
check if output is `init` or `sytemd`

Following commands work to modify processes for systems booted with `init`

```bash
$ sudo service nginx start
 * Starting nginx nginx                                                                                                                                                                                   [ OK ]
```

## To make changes to conf and restart nginx

```bash
# stop service
$ sudo service nginx stop

# make changes to config in editor
$ sudo nano /etc/nginx/sites-available/tsc-p-r.conf

# remove existing sym link
$ sudo rm /etc/nginx/sites-enabled/tsc-p-r.conf

# create new sym link with newly edited conf file
$ sudo ln -s /etc/nginx/sites-available/tsc-p-r.conf /etc/nginx/sites-enabled/

# test for errors
$ sudo nginx -t

# start service again
$ sudo service nginx start
```

---

## Core concepts

### Directives

Everything inside conf file is directive

- Simple Directives

  Consist of directive name followed by space delimited parameter names, terminate with semicolon
  `directive_name param1 param2;`
- Block Directives

  Same as simple directives, end with `{}` instead of `;`
  Include additional instructions in `{}`
  > Block directive capable of conataining other directives inside is called `context`
  > Four `contexts` are
  >
  > 1. events {}\
  > Describe/configure general `handling of requests` on a global level. Max 1 valid.
  > 2. http {}\
  > configure handling of `http, https requests`. Max 1 valid
  > 3. server {}\
  > configure specific `virtual servers` running on `single host`. Nested inside http context. Multiple valid.\
  > __one `server context`__ = __one `virtual host`__
  > 4. main\
  > This is the `conf file` itself.
  > Everything __outside above _three contexts_ is in main__

When .conf file consists of multiple server contexts, `listen` directive is used to _determine which context/instance_ will _handle the incoming requests_

```conf
http {
    server {
        listen 80;
        server_name nginx-handbook.test;

        return 200 "hello from port 80!\n";
    }


    server {
        listen 8080;
        server_name nginx-handbook.test;

        return 200 "hello from port 8080!\n";
    }
}
```

try sending requests to different ports

```bash
curl nginx-handbook.test:80
# hello from port 80!

curl nginx-handbook.test:8080
# hello from port 8080!
```

Change server names

```conf
http {
    server {
        listen 80;
        server_name library.test;

        return 200 "your local library!\n";
    }


    server {
        listen 80;
        server_name librarian.library.test;

        return 200 "welcome dear librarian!\n";
    }
}
```

Now different server instances are running with different servernames
> Update `hosts` file to include the domain names

`return` directive used to respond with valid response to user,\
takes 2 parameters- status code and message to return

### Serve static content

- `/srv` dir in `root` holds site specific data served by system

  cd into this dir and clone your repo

  ```bash
  $ cd /srv

  $ sudo git clone your_repo_path
  ```

- Update .conf file with `root` directive and path to srv/site_content
  > nginx looks for index.html to serve, as it is http server

  ```conf
  events {

  }

  http {

      server {

          listen 80;
          server_name nginx-handbook.test;

          root /srv/nginx-handbook-projects/static-demo;
      }
  }
  ```

  If we check by visiting browser, html is served correctly but css isn't\
  confirm if css is served correctly by making request for the css file

  ```bash
  $ curl -I http://172.28.218.108/mini.min.css
  HTTP/1.1 200 OK
  Server: nginx/1.18.0 (Ubuntu)
  Date: Fri, 03 Feb 2023 09:20:55 GMT
  Content-Type: text/plain
  Content-Length: 46887
  Last-Modified: Fri, 03 Feb 2023 09:12:45 GMT
  Connection: keep-alive
  ETag: "63dcd00d-b727"
  Accept-Ranges: bytes
  ```

  Here we can see the file is served as `text/plain` instead of `text/css`

  Solve this by adding `types` directive to .conf file, and configure file types

  ```conf
  ...
  http {
  ...
    types {
      text/html html;
      text/css css;
    }
  ...
  }
  ```

  Try reloading, if faced with the _mime type error_, we can also `include` the `/etc/nginx/mime.types` file in .conf to resolve this

  ```conf
  events {
  }
  http {
    include /etc/nginx/mime.types;
    server {
        listen 80;
        server_name nginx-handbook.test;

        root /srv/nginx-handbook-projects/static-demo;
    }
  }
  ```

### Location context

### variables

### redirects

### rewrites

### try_files directive
