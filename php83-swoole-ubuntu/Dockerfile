#FROM php:8.3.12-zts-bookworm as pre_base
FROM php:8.3.12-bookworm as pre_base

LABEL maintainer="Zakotnev Vladimir <vladitot@gmail.com>" version="1.0" license="MIT"

ENV DEBIAN_FRONTEND noninteractive

RUN docker-php-ext-install bcmath

RUN apt-get update && apt-get install -y curl libcurl4-openssl-dev libssl-dev libxml2-dev

RUN docker-php-ext-install curl
RUN docker-php-ext-install dom
RUN apt-get update && apt-get install -y libgd-dev zlib1g-dev
RUN docker-php-ext-install gd
RUN apt-get update && apt-get install -y libonig-dev
RUN docker-php-ext-install mbstring
RUN docker-php-ext-install pdo
RUN docker-php-ext-install pdo_mysql
RUN pecl install redis \
    && docker-php-ext-enable redis
RUN docker-php-ext-install simplexml
RUN docker-php-ext-install xml
RUN export EXTRA_CFLAGS="-I/usr/src/php" && docker-php-ext-install xmlreader
RUN docker-php-ext-install xmlwriter
RUN apt-get update && apt-get install -y libzip-dev
RUN docker-php-ext-install zip
RUN apt-get update && apt-get install -y libpq-dev
RUN docker-php-ext-install pgsql
#RUN docker-php-ext-install sqlite3
#RUN docker-php-ext-install mysqli
#RUN docker-php-ext-install pdo_sqlite

RUN apt-get update && apt-get install -y git bash
RUN git clone https://github.com/swoole/swoole-src.git swoole-src
RUN cd swoole-src && \
    git checkout v5.1.5 && \
    phpize && \
    ./configure && \
    make && make install && \
    echo "extension=swoole.so" > /usr/local/etc/php/conf.d/swoole.ini


FROM pre_base as base

ARG CURRENT_UID=1000
ARG CURRENT_GID=1000
RUN addgroup --gid $CURRENT_GID frock
RUN adduser --shell /bin/bash --gid 1000 --disabled-login -u $CURRENT_UID frock

USER frock

FROM base as local
USER root
ENV COMPOSER_VERSION=2.7.8
ENV COMPOSER_ALLOW_SUPERUSER=1
RUN apt-get update && apt-get install -y \
    wget \
    unzip \
    curl \
    git \
    zip \
    nano
RUN wget -nv -O /usr/local/bin/composer https://github.com/composer/composer/releases/download/${COMPOSER_VERSION}/composer.phar \
    && chmod +x /usr/local/bin/composer

RUN pecl install xdebug && docker-php-ext-enable xdebug

COPY 10-xdebug.ini /usr/local/etc/php/conf.d/10-xdebug.ini

ENV PATH="${PATH}:/var/www/php/vendor/bin"

#RUN apt-get install nodejs npm -y

RUN curl -sL https://deb.nodesource.com/setup_20.x -o nodesource_setup.sh
RUN chmod +x nodesource_setup.sh
RUN ./nodesource_setup.sh
RUN apt install nodejs -y

RUN chown -R 1000:1000 /var/www

USER frock
ENTRYPOINT ["php"]
CMD ["-v"]
