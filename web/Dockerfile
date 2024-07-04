FROM php:8.0-apache

## Update package information
RUN apt-get update

#Nano
RUN apt-get install nano
RUN apt-get install unzip

## Configure Apache
RUN a2enmod rewrite \
    && sed -i 's!/var/www/html!/var/www/public!g' /etc/apache2/sites-available/000-default.conf \
    && mv /var/www/html /var/www/public

## Install Composer
RUN curl -sS https://getcomposer.org/installer \
    | php -- --install-dir=/usr/local/bin --filename=composer


###
## PHP Extensions
###

RUN apt-get update
RUN apt-get install -y libfreetype6-dev libjpeg62-turbo-dev libpng-dev

RUN apt-get install -y \
        libzip-dev \
        zip \
  && docker-php-ext-install zip

RUN apt-get install -y libc-client-dev libkrb5-dev &&     rm -r /var/lib/apt/lists/*
RUN docker-php-ext-configure imap --with-kerberos --with-imap-ssl
RUN docker-php-ext-install imap


RUN docker-php-ext-install calendar
RUN docker-php-ext-configure gd
RUN docker-php-ext-install -j$(nproc) gd

RUN docker-php-ext-enable gd

RUN echo -n /usr/local/etc/php/conf.d/docker-php-ext-gd.ini
RUN echo "extension=gd.so" > /usr/local/etc/php/conf.d/docker-php-ext-gd.ini

## xDebug
RUN pecl install xdebug
RUN docker-php-ext-enable xdebug
RUN echo -n /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "zend_extension=/usr/local/lib/php/extensions/no-debug-non-zts-20190902/xdebug.so" > /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "xdebug.mode=coverage" > /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

## Install intl library and extension
RUN apt-get update && apt-get install --yes libicu-dev
RUN docker-php-ext-configure intl
RUN docker-php-ext-install intl

## MySQL PDO support
RUN docker-php-ext-install pdo_mysql

RUN docker-php-ext-install mysqli

RUN apt-get update && apt-get install --yes mariadb-client


WORKDIR /var/www/html
