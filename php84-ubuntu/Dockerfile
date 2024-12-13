FROM ubuntu:22.04 as pre_base

LABEL maintainer="Zakotnev Vladimir <vladitot@gmail.com>" version="1.0" license="MIT"

ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update
RUN apt-get install -y software-properties-common ca-certificates apt-transport-https software-properties-common lsb-release
RUN add-apt-repository ppa:ondrej/php -y
RUN apt-get update
RUN apt-get install -y \
    php8.4

RUN apt-get update && apt-get install -y \
    php8.4-bcmath \
    php8.4-curl \
    php8.4-dom \
    php8.4-gd \
    php8.4-mbstring \
    php8.4-mysqlnd \
    php8.4-pdo \
    php8.4-mysql \
    php8.4-redis \
    php8.4-simplexml \
    php8.4-tokenizer \
    php8.4-xml \
    php8.4-xmlreader \
    php8.4-xmlwriter \
    php8.4-zip \
    php8.4-dev \
    php8.4-pgsql \
    php8.4-sqlite

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

RUN pecl install mongodb \
    &&  echo "extension=mongodb.so" > /etc/php/8.4/cli/conf.d/mongo.ini

RUN pecl install dio \
    &&  echo "extension=dio.so" > /etc/php/8.4/cli/conf.d/dio.ini

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
ENV COMPOSER_VERSION=2.8.4
ENV COMPOSER_ALLOW_SUPERUSER=1

RUN mkdir -p /.devspace
RUN chmod 777 /.devspace

RUN wget -nv -O /usr/local/bin/composer https://github.com/composer/composer/releases/download/${COMPOSER_VERSION}/composer.phar \
    && chmod +x /usr/local/bin/composer

RUN apt-get install -y php8.4-xdebug

COPY 10-xdebug.ini /etc/php/8.4/cli/conf.d/10-xdebug.ini

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
