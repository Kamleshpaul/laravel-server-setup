
# Server Setup for Laravel

Follow the steps below to set up your Ubuntu 22.0 server.

## Initial Server Setup

1. First, ensure your server's package repositories are up to date:
```sh
apt update && apt upgrade && apt autoremove
```

## Installing Necessary Packages

2. We're going to install a range of packages including PHP, Node.js, npm, Yarn, MySQL, and Nginx. Run the following commands:

```sh
apt-get update \
&& apt-get install -y build-essential libssl-dev curl zip unzip git supervisor sqlite3 libcap2-bin libpng-dev python2 dnsutils librsvg2-bin fswatch \
&& curl -sS 'https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x14aa40ec0831756756d7f66c4f4ea0aae5267a6c' | gpg --dearmor | tee /etc/apt/keyrings/ppa_ondrej_php.gpg > /dev/null \
&& echo "deb [signed-by=/etc/apt/keyrings/ppa_ondrej_php.gpg] https://ppa.launchpadcontent.net/ondrej/php/ubuntu jammy main" > /etc/apt/sources.list.d/ppa_ondrej_php.list \
&& apt-get update \
&& apt-get install -y php8.2-cli php8.2-dev php8.2-fpm\
    php8.2-sqlite3 php8.2-gd php8.2-imagick \
    php8.2-curl \
    php8.2-imap php8.2-mysql php8.2-mbstring \
    php8.2-xml php8.2-zip php8.2-bcmath php8.2-soap \
    php8.2-intl php8.2-readline \
    php8.2-ldap \
    php8.2-msgpack php8.2-igbinary php8.2-redis php8.2-swoole \
    php8.2-memcached php8.2-pcov php8.2-xdebug \
&& curl -sLS https://getcomposer.org/installer | php -- --install-dir=/usr/bin/ --filename=composer \
&& apt-get update \
&& curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash \
&& source ~/.nvm/nvm.sh \
&& nvm install --lts \
&& nvm use --lts \
&& npm install -g npm \
&& curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | gpg --dearmor | tee /etc/apt/keyrings/yarn.gpg >/dev/null \
&& echo "deb [signed-by=/etc/apt/keyrings/yarn.gpg] https://dl.yarnpkg.com/debian/ stable main" > /etc/apt/sources.list.d/yarn.list \
&& apt-get update \
&& apt-get install -y yarn \
&& apt-get install -y mysql-server \
&& apt-get install -y nginx \
&& apt-get -y autoremove \
&& apt-get clean \
&& rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* \
&& sudo mysql_secure_installation
```

## Configuring Nginx

3. Move to the Nginx configuration directory:
```sh
cd /etc/nginx/sites-enabled
```

4. Create and edit a new configuration file for your domain:
```sh
sudo nano domain.conf
```

5. Copy and paste the following configuration into the file. Don't forget to replace `example.com` with your domain name:

```sh
server {
    listen 80;
    listen [::]:80;
    server_name example.com;
    root /srv/example.com/public;
 
    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";
 
    index index.php;
 
    charset utf-8;
 
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    # Add WebSocket configuration here
    #location /app/app-key {
    #   proxy_pass http://localhost:6001;
    #    proxy_http_version 1.1;
    #   proxy_set_header Upgrade $http_upgrade;
    #    proxy_set_header Connection "Upgrade";
    #    proxy_set_header Host $host;
    #}

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }
 
    error_page 404 /index.php;
 
    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }
 
    location ~ /\.(?!well-known).* {
        deny all;
    }

    client_max_body_size 75M;
}
```

6. Test your Nginx configuration:
```sh
nginx -t
```
If the output indicates success (`nginx: configuration file /etc/nginx/nginx.conf test is successful`), proceed to the next step.

7. Reload Nginx to apply the changes:
```sh
service nginx reload
```

## Setting Up SSL with Certbot

8. To secure your site with HTTPS, we'll use Certbot. Run the following commands to install and configure Certbot for Nginx:

```sh
(Your existing certbot commands here)
```

## Setting Up Cron for Laravel Tasks

9. Open the crontab editor to schedule tasks:
```sh
sudo crontab -e
```

10. Add the following line to run Laravel's scheduled tasks every minute. Replace `/srv/example.com` with your project's path if different:

```sh
* * * * * cd /srv/example.com && php artisan schedule:run >> /dev/null 2>&1
```

---

**Note**: Ensure you've backed up any existing configurations or data before making changes. Always test configurations in a safe environment before applying them to a production server.

