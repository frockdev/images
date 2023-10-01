ARG PHP_VERSION=8.2
ARG LOCAL="local"

ARG BASE="base"

FROM php:${PHP_VERSION}-cli-buster as base

ENV TZ=Europe/Moscow
ENV DEBIAN_FRONTEND=noninteractive
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN apt-get update && apt-get install -y gcc make g++ curl librabbitmq-dev
RUN apt-get install -y ca-certificates \
    tzdata \
    vim \
    nano
RUN apt-get install -y wget \
    unzip \
    git \
    bash \
    autoconf
RUN apt-get install -y libpng-dev libzip-dev libxml2-dev libpq-dev libcurl4-openssl-dev \
    libcurl4 libonig-dev libssl-dev gnupg gnupg1 gnupg2


RUN curl -L https://github.com/roadrunner-server/roadrunner/releases/download/v2023.1.2/roadrunner-2023.1.2-linux-amd64.tar.gz | tar -xz
RUN mv roadrunner-2023.1.2-linux-amd64/rr /usr/local/bin/rr && chmod +x /usr/local/bin/rr && rm -rf roadrunner-2023.1.2-linux-amd64

RUN apt-get install libyaml-dev -y
RUN  pecl install yaml && echo "extension=yaml.so" > /usr/local/etc/php/conf.d/ext-yaml.ini && docker-php-ext-enable yaml
RUN pecl install amqp-1.11.0 && echo "extension=amqp.so" > /usr/local/etc/php/conf.d/ext-amqp.ini && docker-php-ext-enable amqp

RUN docker-php-ext-configure intl \
    && docker-php-ext-configure pdo \
    && docker-php-ext-configure pdo_mysql \
    && docker-php-ext-configure pdo_pgsql \
    && docker-php-ext-configure phar \
    && docker-php-ext-configure xml \
    && docker-php-ext-configure curl \
    && docker-php-ext-configure mbstring \
    && docker-php-ext-configure opcache \
    && docker-php-ext-configure zip \
    && docker-php-ext-configure gd \
    && docker-php-ext-configure bcmath \
    && docker-php-ext-configure soap \
    && docker-php-ext-configure pcntl \
    && docker-php-ext-configure calendar \
    && docker-php-ext-configure sockets \
    && docker-php-ext-configure mysqli \
    && docker-php-ext-configure pgsql \
#######
    && docker-php-ext-install intl \
    && docker-php-ext-install pdo \
    && docker-php-ext-install pdo_mysql \
    && docker-php-ext-install pdo_pgsql \
    && docker-php-ext-install phar \
    && docker-php-ext-install xml \
    && docker-php-ext-install curl \
    && docker-php-ext-install mbstring \
    && docker-php-ext-install opcache \
    && docker-php-ext-install zip \
    && docker-php-ext-install gd \
    && docker-php-ext-install bcmath \
    && docker-php-ext-install soap \
    && docker-php-ext-install pcntl \
    && docker-php-ext-install calendar \
    && docker-php-ext-install sockets \
    && docker-php-ext-install mysqli \
    && docker-php-ext-install pgsql

RUN pecl update-channels && pecl install grpc
RUN docker-php-ext-enable grpc

RUN pecl update-channels && pecl install -o -f redis pcov apcu \
    &&  rm -rf /tmp/pear \
    &&  docker-php-ext-enable redis pcov apcu


ENV PATH $PATH:/var/www/vendor/bin:/var/www/bin

RUN php --ini

COPY php.ini /usr/local/etc/php/php.ini

WORKDIR /var/www

RUN echo PATH="$PATH:/var/www/vendor/bin" >> /etc/profile
RUN echo 'alias artisan="php /var/www/artisan"' >> /etc/profile

ARG CURRENT_UID=1000
ARG CURRENT_GID=1000
RUN addgroup --gid $CURRENT_GID professional2 || true
RUN adduser --shell /bin/bash --disabled-login --gid $CURRENT_GID --uid $CURRENT_UID professional2

RUN mkdir -p /home/professional2/.config/psysh && chown -R professional2 /home/professional2/
RUN chmod -R 777 /var/www

RUN apt-get update && apt-get install -y bc
RUN wget -qO /usr/local/bin/yq https://github.com/mikefarah/yq/releases/latest/download/yq_linux_amd64
RUN chmod a+x /usr/local/bin/yq

ENTRYPOINT ["rr"]
CMD [ "serve", "-d", "-c", "/var/.rr.yaml", "-w", "/var/www" ]

FROM ${BASE} as local
ARG CURRENT_UID=1000
ARG CURRENT_GID=1000
USER root

RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
RUN chmod +x /usr/local/bin/composer

RUN pecl install xdebug-3.2.2 \
    && docker-php-ext-enable xdebug

COPY 10-xdebug.ini /usr/local/etc/php/conf.d/10-xdebug.ini
USER $CURRENT_UID

COPY ssh_config /etc/ssh/ssh_config

RUN curl -fsSL https://deb.nodesource.com/setup_16.x | bash -
RUN apt-get install -y nodejs

USER $CURRENT_UID

ENTRYPOINT ["rr"]
CMD [ "serve", "-d", "-c", "/var/.rr.yaml", "--debug", "-w", "/var/www" ]