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


# hyperf/hyperf:8.2
#
# @link     https://www.hyperf.io
# @document https://hyperf.wiki
# @contact  group@hyperf.io
# @license  https://github.com/hyperf/hyperf/blob/master/LICENSE

FROM pre_base as base

LABEL maintainer="Hyperf Developers <group@hyperf.io>" version="1.0" license="MIT"

ARG SW_VERSION='v1.3.1'
ARG COMPOSER_VERSION='2.6.5'
ARG PHP_BUILD_VERSION='82'

##
# ---------- env settings ----------
##
ENV SW_VERSION=${SW_VERSION:-"develop"} \
    COMPOSER_VERSION=${COMPOSER_VERSION:-"2.0.2"} \
    COMPOSER_ALLOW_SUPERUSER=1 \
    #  install and remove building packages
    PHPIZE_DEPS="autoconf automake gcc g++ make php${PHP_BUILD_VERSION}-dev php${PHP_BUILD_VERSION}-pear file re2c openssl-dev curl-dev"

# update
RUN set -ex \
    && apk update \
    # for swow extension libaio linux-headers
    && apk add --no-cache openssl git bash \
    && apk add --no-cache --virtual .build-deps $PHPIZE_DEPS \
    # download
    && cd /tmp \
    && curl -SL "https://github.com/swow/swow/archive/${SW_VERSION}.tar.gz" -o swow.tar.gz \
    && ls -alh \
    # php extension:swow
    && cd /tmp \
    && mkdir -p swow \
    && tar -xf swow.tar.gz -C swow --strip-components=1 \
    && ln -s /usr/bin/phpize${PHP_BUILD_VERSION} /usr/local/bin/phpize \
    && ln -s /usr/bin/php-config${PHP_BUILD_VERSION} /usr/local/bin/php-config \
    && ( \
        cd swow/ext \
        && phpize \
        && ./configure --enable-swow --enable-swow-ssl --enable-swow-curl \
        && make -s -j$(nproc) && make install \
    ) \
    && echo "memory_limit=1G" > /etc/php${PHP_BUILD_VERSION}/conf.d/00_default.ini \
    && echo "opcache.enable_cli = 'On'" >> /etc/php${PHP_BUILD_VERSION}/conf.d/00_opcache.ini \
    && echo "extension=swow.so" > /etc/php${PHP_BUILD_VERSION}/conf.d/50_swow.ini \
    # install composer
    && wget -nv -O /usr/local/bin/composer https://github.com/composer/composer/releases/download/${COMPOSER_VERSION}/composer.phar \
    && chmod u+x /usr/local/bin/composer \
    # php info
    && php -v \
    && php -m \
    && php --ri swow \
    && composer \
    # ---------- clear works ----------
    && apk del .build-deps \
    && rm -rf /var/cache/apk/* /tmp/* /usr/share/man /usr/local/bin/php* \
    && echo -e "\033[42;37m Build Completed :).\033[0m\n"

FROM prod_env as dev_env

