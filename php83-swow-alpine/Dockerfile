ARG NODE_VERSION=20.17.0
ARG ALPINE_VERSION=v3.20
ARG SWOW_VERSION=v1.5.3
ARG PHP_VERSION=8.3

FROM node:${NODE_VERSION}-alpine AS node

FROM hyperf/hyperf:${PHP_VERSION}-alpine-${ALPINE_VERSION}-swow-${SWOW_VERSION} as pre_base

RUN apk --no-cache --repository https://dl-cdn.alpinelinux.org/alpine/v3.20/main add \
    icu-libs \
    && apk --no-cache --repository https://dl-cdn.alpinelinux.org/alpine/v3.20/community add \
    # Current packages don't exist in other repositories
    libavif \
    gnu-libiconv \
    # Packages \
    php83-dev \
    php83-bcmath \
    php83-curl \
    php83-dom \
    php83-gd \
    php83-mbstring \
    php83-mysqli \
    php83-pdo \
    php83-redis \
    php83-simplexml \
    php83-tokenizer \
    php83-xml \
    php83-xmlreader \
    php83-xmlwriter \
    php83-zip \
    php83-pgsql \
    php83-sqlite3 \
    php83-pecl-mongodb
#RUN pecl install dio
#RUN pecl install mongodb

FROM pre_base as base

RUN addgroup -g 1000 frock
RUN adduser -D -s /bin/bash -G frock -u 1000 frock

USER 1000

FROM base as local

USER root
ENV COMPOSER_VERSION=2.7.8
ENV COMPOSER_ALLOW_SUPERUSER=1

RUN wget -nv -O /usr/local/bin/composer https://github.com/composer/composer/releases/download/${COMPOSER_VERSION}/composer.phar \
    && chmod +x /usr/local/bin/composer

RUN apk add -U --no-cache --repository http://dl-cdn.alpinelinux.org/alpine/v3.20/testing/   \
        wget \
        unzip \
        curl \
        git \
        zip \
        php83-xdebug \
        nano \
    # Delete APK cache.
    && rm -rf /var/cache/apk/*

COPY --from=node /usr/lib /usr/lib
COPY --from=node /usr/local/lib /usr/local/lib
COPY --from=node /usr/local/include /usr/local/include
COPY --from=node /usr/local/bin /usr/local/bin

COPY 10-xdebug.ini /etc/php83/conf.d/10-xdebug.ini

USER 1000