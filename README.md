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
