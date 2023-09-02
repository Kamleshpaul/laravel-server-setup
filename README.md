## Server Setup

- Update server

```sh
sudo apt update
sudo apt upgrade
sudo apt autoremove
```
---

* install `php 8.0` 

```sh
sudo apt install software-properties-common
sudo add-apt-repository ppa:ondrej/php
sudo apt update 
sudo apt install -y  php8.0-cli php8.0-fpm php8.0-gd php8.0-curl php8.0-mysql php8.0-mbstring php8.0-xml php8.0-zip php8.0-bcmath php8.0-soap
```


---
* insall `mysql`

```sh 
sudo apt install mysql-server

sudo mysql_secure_installation
sudo mysql -uroot -p<password>
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'dragoncloud';
mysql> FLUSH PRIVILEGES;

```
---

* install `composer`
```sh
sudo php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"

sudo php -r "if (hash_file('sha384', 'composer-setup.php') === '756890a4488ce9024fc62c56153228907f1545c228516cbf63f885e036d37e9a59d27d63f46af1d4d07ee0f76181c7d3') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"

sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer

php -r "unlink('composer-setup.php');"

```
>NOTE:- if it outdated use https://getcomposer.org/download/
---
* install `nginx`
```sh
sudo apt install nginx

cd /etc/nginx/sites-enabled

sudo nano domain.conf
```

put this code 

```sh
server {
    
    listen 80;
    server_name domain;
    root /var/www/html/domain/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Content-Type-Options "nosniff";

    index index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.0-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }

    client_max_body_size 75M;
}
```
