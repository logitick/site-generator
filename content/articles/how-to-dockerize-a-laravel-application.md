---
title: "How to dockerize a Laravel application"
date: 2018-09-15T21:31:04+08:00
draft: true
---
First off, why would we need to do that? If it ain't broke, right?  

Here are some benefits just at the top of my head:  
**Consistency**  -  when you deploy the image to any machine, you're sure that it 
behaves exactly the same way. It will run on any OS that supports docker. It will run with the exact 
same dependencies that were present when building the image.

**Isolation ** - It's one of the selling points of containerization. You can
 rm -rf /all you want. You can delete everything in the container but you 
can rest assured that everything in the Docker host remains intact. Run the 
image back again and everything is back to the state right after it was built.

But I digress. You're already here and you must be already convinced that 
Docker is the right tool for your project otherwise you can read about it 
in the Docker website: [Why Docker](https://www.docker.com/why-docker).

The Docker image we will be building will match the conventional LAMP 
setup. It is recommended that there only be one processing running 
in a container so we'll leave out MySql.

I'll assume that you already have Docker installed in your system. 
If not have a look at their website [https://docs.docker.com/install/](https://docs.docker.com/install/).

## Dockerfile
We'll start by creating a file named `Dockerfile` in the root directory 
of your Laravel application. This file is required for building your
Docker images. 

Think of a Docker image as a `class` in an application. A container 
in this metaphore is an instance of that `class`.

You should build on official Docker images as much as you can. The [official
PHP image](https://hub.docker.com/_/php/) should be safe to use in production systems.
This will be the first line of our `Dockerfile`:
```
FROM php:7.2.10-apache # This indicates that we will be extending the official pre-built Docker image.
```
One of the things that I love about Docker is that you can make your 
containers run any Linux distro you want. SInce we're extending a Docker 
image that runs Debian we can install packages with `apt`:
```
RUN apt-get update && apt-get install -y \
        libcurl4-openssl-dev \
        pkg-config libssl-dev \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libmcrypt-dev \
        libpng-dev \
     && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
     && docker-php-ext-install -j$(nproc) gd pdo pdo_mysql
```
Explain above snippet here.

## Apache configuration
You'll need to add a file named `000-default.conf` to the laravel application's root directory.
This will be the virtualhost configuration that will be added to the Apache instance be running in the Docker.
```
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    # laravel root directory should be in /var/www/html
    DocumentRoot /var/www/html/public

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

    <Directory "/var/www/html/public">
        AllowOverride all
    </Directory>
</VirtualHost>

```

```
FROM composer:1.7 as builder

RUN mkdir -p /var/www/html
WORKDIR /var/www/html

ADD composer.json ./
ADD composer.lock ./
ADD database/ database/
RUN composer install --no-scripts --no-autoloader --no-dev

FROM php:7.1.3-apache

RUN apt-get update && apt-get install -y \
        libcurl4-openssl-dev \
        pkg-config libssl-dev \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libmcrypt-dev \
        libpng-dev \
     && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
     && docker-php-ext-install -j$(nproc) gd pdo pdo_mysql

WORKDIR /var/www/html
RUN curl -o composer https://getcomposer.org/download/1.7.2/composer.phar && chmod +x composer

ADD . /var/www/html
COPY --from=builder /var/www/html/vendor/ ./vendor/

RUN ./composer dump-autoload --optimize --no-dev  && rm composer
RUN chown www-data:www-data -R * && chmod -Rv ug+rwx storage bootstrap/cache
ADD 000-default.conf /etc/apache2/sites-available/000-default.conf
RUN a2enmod rewrite


```
