# install word press on amazon-linux

- ### install nginx
```
    sudo yum install nginx
```

- ### install php8.2

```
- sudo yum install php8.2 php8.2-cli php8.2-fpm php8.2-mysqlnd  php8.2-xml php8.2-opcache php8.2-mbstring php8.2-zip
```

- ### edit php-fpm config

```
- sudo vim /etc/php-fpm/www.conf
....
user = nginx
...
group = nginx
...
listen = /run/php-fpm/www.sock
...
listen.owner = nginx
listen.group = nginx
```

- ### install mysql

```
- sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm
- sudo yum install mysql80-community-release-el9-1.noarch.rpm
- sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
- sudo yum install mysql-community-server
- sudo systemctl enable mysqld
- sudo systemctl start mysqld
```

- ### change root password and create wordpress_user

```
- cat /var/log/mysqld.log | grep password
- mysql -u root -p
- ALTER USER 'root'@'localhost' IDENTIFIED BY 'strong_password'
- create data: create database wordpress
```

- ### create user and grant permission
```
- CREATE USER 'wordpress_user'@'%' IDENTIFIED BY 'Ub31cBmrdv@';

- GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpress_user'@'%';
```

- ## config nginx

- ### create configuration folder:

```
    - mkdir /etc/nginx/sites-available
    - mkdir /etc/nginx/sites-enabled
```

- ### copy file:

```
    - wordpress.conf
    - php-fpm.conf
    - tuning.conf
```

- #### make symbolic link:

```
    - ln -s /etc/nginx/sites-available/wordpress.conf /etc/nginx/sites-enabled/
    - ln -s /etc/nginx/sites-available/php-fpm.conf /etc/nginx/sites-enabled/
    - ln -s /etc/nginx/sites-available/tuning.conf /etc/nginx/sites-enabled/
```

- ### import config:

```
   - vim /etc/nginx/nginx.conf

    ...
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*.conf;
    ....
```

- ### download wordpress

```
- wget https://wordpress.org/wordpress-6.5.2.zip

- unzip wordpress-6.5.2.zip

- mv wordpress /var/www

```

- ### enable and start service

```
    sudo systemctl enable php-fpm
    sudo systemctl start php-fpm
    sudo systemctl start nginx
    sudo systemctl enable nginx
```

# self sign certificate

```
  openssl req -x509 -newkey rsa:4096 -keyout server.key -out server.cert -sha256 -days 3650 -nodes -subj "/C=VN/ST=HaNoi/L=HaNoi/O=NoCompany/OU=infra/CN=wordpress.duongdx.com"
```


# Config ssl/tls with let's encrypt

- ### install let's encrypt

```
sudo dnf install -y augeas-libs
sudo python3 -m venv /opt/certbot/
sudo /opt/certbot/bin/pip install --upgrade pip
sudo /opt/certbot/bin/pip install certbot certbot-nginx
```

- ### create cert:

```
sudo /opt/certbot/bin/certbot certonly --nginx -d duongdx.com
```

- ### verify certificate

```
sudo /opt/certbot/bin/certbot certificates
```

- ### testing renew

```
sudo /opt/certbot/bin/certbot renew --dry-run
```

- ### auto renew cert

```
- crontab -e
- 0 0 * * * /opt/certbot/bin/certbot renew >> /var/log/letsencrypt-renewal.log
```
