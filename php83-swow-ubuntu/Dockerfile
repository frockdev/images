FROM ubuntu:22.04 as pre_base

LABEL maintainer="Zakotnev Vladimir <vladitot@gmail.com>" version="1.0" license="MIT"

ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update
RUN apt-get install -y software-properties-common ca-certificates apt-transport-https software-properties-common lsb-release
RUN add-apt-repository ppa:ondrej/php -y
RUN apt-get update
RUN apt-get install -y \
    php8.3

RUN apt-get update && apt-get install -y \
    php8.3-bcmath \
    php8.3-curl \
    php8.3-dom \
    php8.3-gd \
    php8.3-mbstring \
    php8.3-mysqlnd \
    php8.3-pdo \
    php8.3-mysql \
    php8.3-redis \
    php8.3-simplexml \
    php8.3-tokenizer \
    php8.3-xml \
    php8.3-xmlreader \
    php8.3-xmlwriter \
    php8.3-zip \
    php8.3-dev \
    php8.3-pgsql \
    php8.3-sqlite

RUN apt-get install -y \
    curl \
    wget \
    tar \
    tzdata \
    xz-utils \
    libpcre3 libpcre3-dev


RUN apt-get install -y \
    openssl git bash

RUN apt-get -y install build-essential autoconf libtool bison re2c pkg-config

ENV SW_VERSION=v1.5.3
# update
RUN set -ex \
    && apt-get update \
    # for swow extension libaio linux-headers
    && apt-get install -y openssl git bash \
    # download
    && cd /tmp \
    && curl -SL "https://github.com/swow/swow/archive/${SW_VERSION}.tar.gz" -o swow.tar.gz \
    && cd /tmp \
    && mkdir -p swow \
    && tar -xf swow.tar.gz -C swow --strip-components=1 \
    && ln -s /usr/bin/phpize8.3 /usr/local/bin/phpize \
    && ln -s /usr/bin/php-config8.3 /usr/local/bin/php-config \
    && ( \
        cd swow/ext \
        && phpize \
        && ./configure --enable-swow --enable-swow-ssl --enable-swow-curl \
        && make -s -j$(nproc) && make install \
    ) \
    && echo "memory_limit=1G" > /etc/php/8.3/cli/conf.d/00_default.ini \
    && echo "max_input_vars=PHP_INT_MAX" >> /etc/php/8.3/cli/conf.d/00_default.ini \
    && echo "opcache.enable_cli = 'On'" >> /etc/php/8.3/cli/conf.d/00_opcache.ini \
    && echo "extension=swow.so" > /etc/php/8.3/cli/conf.d/50_swow.ini

RUN pecl install mongodb \
    &&  echo "extension=mongodb.so" > /etc/php/8.3/cli/conf.d/mongo.ini

RUN pecl install dio \
    &&  echo "extension=dio.so" > /etc/php/8.3/cli/conf.d/dio.ini

FROM ghcr.io/roadrunner-server/roadrunner:2024.2.0 AS roadrunner
FROM pre_base as base

ARG CURRENT_UID=1000
ARG CURRENT_GID=1000
RUN addgroup --gid $CURRENT_GID frock
RUN adduser --shell /bin/bash --gid 1000 --disabled-login -u $CURRENT_UID frock
RUN apt-get install -y unzip
COPY --from=roadrunner /usr/bin/rr /usr/local/bin/rr

USER frock


FROM base as local
USER root
ENV COMPOSER_VERSION=2.7.8
ENV COMPOSER_ALLOW_SUPERUSER=1

RUN mkdir -p /.devspace
RUN chmod 777 /.devspace

RUN wget -nv -O /usr/local/bin/composer https://github.com/composer/composer/releases/download/${COMPOSER_VERSION}/composer.phar \
    && chmod +x /usr/local/bin/composer

RUN apt-get install -y php8.3-xdebug

COPY 10-xdebug.ini /etc/php/8.3/cli/conf.d/10-xdebug.ini

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
