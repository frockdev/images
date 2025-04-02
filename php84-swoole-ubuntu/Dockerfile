FROM phpswoole/swoole:6.0.0-php8.4-zts as base

    # Install dependencies
RUN apt-get update && apt-get install -y \
    libcurl4-openssl-dev \
    libssl-dev \
    libxml2-dev \
    libgd-dev \
    zlib1g-dev \
    libonig-dev \
    libzip-dev \
    libpq-dev \
    && apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Install PHP extensions
RUN docker-php-ext-install bcmath
RUN docker-php-ext-install curl
RUN docker-php-ext-install dom
RUN docker-php-ext-install gd
RUN docker-php-ext-install mbstring
RUN docker-php-ext-install pdo
RUN docker-php-ext-install pdo_mysql
RUN docker-php-ext-install pdo_pgsql
RUN docker-php-ext-install simplexml
RUN docker-php-ext-install xml
RUN docker-php-ext-install xmlwriter
RUN docker-php-ext-install zip
RUN docker-php-ext-install pgsql
RUN docker-php-ext-install mysqli
RUN docker-php-ext-install pdo_sqlite

FROM base as local

# Install xdebug
RUN apt-get update && apt-get install -y \
    git \
    unzip \
    && apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Install composer
COPY --from=composer:2.8.6 /usr/bin/composer /usr/bin/composer

FROM local as local-xdebug

RUN pecl install xdebug-3.4.2

RUN echo "zend_extension=xdebug" > /usr/local/etc/php/conf.d/xdebug.ini
RUN echo "xdebug.mode=develop,debug,coverage" >> /usr/local/etc/php/conf.d/xdebug.ini
RUN echo "xdebug.client_host=host.docker.internal" >> /usr/local/etc/php/conf.d/xdebug.ini
RUN echo "xdebug.discover_client_host=0" >> /usr/local/etc/php/conf.d/xdebug.ini
RUN echo "xdebug.start_with_request=yes" >> /usr/local/etc/php/conf.d/xdebug.ini
RUN echo "xdebug.idekey=PHPSTORM" >> /usr/local/etc/php/conf.d/xdebug.ini