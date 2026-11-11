FROM php:8.5-fpm-alpine

# https://github.com/mlocati/docker-php-extension-installer
COPY --from=mlocati/php-extension-installer:2 /usr/bin/install-php-extensions /usr/local/bin/

RUN install-php-extensions \
        pdo_mysql mysqli gd zip intl opcache bcmath exif redis imagick \
    && apk upgrade --no-cache && apk add --no-cache tzdata

COPY --from=composer:2 /usr/bin/composer /usr/local/bin/composer

COPY php.ini /usr/local/etc/php/conf.d/zz-custom.ini

WORKDIR /var/www
