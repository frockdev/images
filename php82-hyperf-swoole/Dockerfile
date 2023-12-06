# hyperf/hyperf:8.1
#
# @link     https://www.hyperf.io
# @document https://hyperf.wiki
# @contact  group@hyperf.io
# @license  https://github.com/hyperf/hyperf/blob/master/LICENSE


FROM alpine:3.18 as pre_base

LABEL maintainer="Hyperf Developers <group@hyperf.io>" version="1.0" license="MIT"

##
# ---------- building ----------
##
COPY ./init.php /init.php

RUN set -ex \
    && apk update \
    && apk add --no-cache \
    # Install base packages ('ca-certificates' will install 'nghttp2-libs')
    ca-certificates \
    curl \
    wget \
    tar \
    xz \
    tzdata \
    pcre \
    php82 \
    php82-bcmath \
    php82-curl \
    php82-ctype \
    php82-dom \
    php82-fileinfo \
    php82-gd \
    php82-iconv \
    php82-mbstring \
    php82-mysqlnd \
    php82-openssl \
    php82-pdo \
    php82-pdo_mysql \
    php82-pdo_sqlite \
    php82-phar \
    php82-posix \
    php82-redis \
    php82-sockets \
    php82-sodium \
    php82-sysvshm \
    php82-sysvmsg \
    php82-sysvsem \
    php82-simplexml \
    php82-tokenizer \
    php82-zip \
    php82-zlib \
    php82-xml \
    php82-xmlreader \
    php82-xmlwriter \
    php82-pcntl \
    php82-opcache \
    && ln -sf /usr/bin/php82 /usr/bin/php \
    && php /init.php \
    && apk del --purge *-dev \
    && rm -rf /var/cache/apk/* /tmp/* /usr/share/man /usr/share/php82 \
    && php -v \
    && php -m \
    && echo -e "\033[42;37m Build Completed :).\033[0m\n"
RUN apk add --no-cache --repository=http://dl-cdn.alpinelinux.org/alpine/edge/community php82-pecl-grpc grpc \
        && apk del --purge *-dev \
        && rm -rf /var/cache/apk/* /tmp/* /usr/share/man /usr/share/php82

# hyperf/hyperf:8.2
#
# @link     https://www.hyperf.io
# @document https://hyperf.wiki
# @contact  group@hyperf.io
# @license  https://github.com/hyperf/hyperf/blob/master/LICENSE

FROM pre_base as base

ARG SW_VERSION
ARG COMPOSER_VERSION
ARG PHP_BUILD_VERSION='82'

##
# ---------- env settings ----------
##
ENV SW_VERSION=${SW_VERSION:-"v5.1.1"} \
    COMPOSER_VERSION=${COMPOSER_VERSION:-"2.6.5"} \
    COMPOSER_ALLOW_SUPERUSER=1 \
    #  install and remove building packages
    PHPIZE_DEPS="autoconf dpkg-dev dpkg file g++ gcc libc-dev make php${PHP_BUILD_VERSION}-dev php${PHP_BUILD_VERSION}-pear pkgconf re2c pcre-dev pcre2-dev zlib-dev libtool automake libaio-dev openssl-dev curl-dev"

# update
RUN set -ex \
    && apk update \
    # for swoole extension libaio linux-headers
    && apk add --no-cache libstdc++ openssl git bash c-ares-dev libpq-dev \
    && apk add --no-cache --virtual .build-deps $PHPIZE_DEPS \
    # download
    && cd /tmp \
    && curl -SL "https://github.com/swoole/swoole-src/archive/${SW_VERSION}.tar.gz" -o swoole.tar.gz \
    && ls -alh \
    # php extension:swoole
    && cd /tmp \
    && mkdir -p swoole \
    && tar -xf swoole.tar.gz -C swoole --strip-components=1 \
    && ln -s /usr/bin/phpize${PHP_BUILD_VERSION} /usr/local/bin/phpize \
    && ln -s /usr/bin/php-config${PHP_BUILD_VERSION} /usr/local/bin/php-config \
    && ( \
        cd swoole \
        && phpize \
        && ./configure --enable-openssl --enable-swoole-curl --enable-cares --enable-swoole-pgsql \
        && make -s -j$(nproc) && make install \
    ) \
    && echo "memory_limit=1G" > /etc/php${PHP_BUILD_VERSION}/conf.d/00_default.ini \
    && echo "opcache.enable_cli = 'On'" >> /etc/php${PHP_BUILD_VERSION}/conf.d/00_opcache.ini \
    && echo "extension=swoole.so" > /etc/php${PHP_BUILD_VERSION}/conf.d/50_swoole.ini \
    && echo "swoole.use_shortname = 'Off'" >> /etc/php${PHP_BUILD_VERSION}/conf.d/50_swoole.ini \
    # install composer
    && wget -nv -O /usr/local/bin/composer https://github.com/composer/composer/releases/download/${COMPOSER_VERSION}/composer.phar \
    && chmod u+x /usr/local/bin/composer \
    # ---------- clear works ----------
    && apk del .build-deps \
    && rm -rf /var/cache/apk/* /tmp/* /usr/share/man /usr/local/bin/php* \
    # php info
    && php -v \
    && php -m \
    && php --ri swoole \
    && php --ri Zend\ OPcache \
    && composer \
    && echo -e "\033[42;37m Build Completed :).\033[0m\n"

ARG CURRENT_UID=1000
ARG CURRENT_GID=1000
RUN addgroup --gid $CURRENT_GID frock
RUN adduser --shell /bin/bash -G frock -D -u $CURRENT_UID frock
RUN echo "grpc.enable_fork_support = 1" >> /etc/php82/php.ini

FROM base as local

RUN apk add --no-cache php82-pecl-xdebug

COPY 10-xdebug.ini /etc/php${PHP_BUILD_VERSION}/conf.d/10-xdebug.ini
