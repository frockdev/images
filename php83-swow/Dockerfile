FROM hyperf/hyperf:8.3-alpine-vedge-swow as pre_base


FROM pre_base as base
USER root
RUN apk add --no-cache --repository=http://dl-cdn.alpinelinux.org/alpine/edge/community php83-pecl-grpc grpc \
        && apk del --purge *-dev \
        && rm -rf /var/cache/apk/* /tmp/* /usr/share/man /usr/share/php83

ARG CURRENT_UID=1000
ARG CURRENT_GID=1000
RUN addgroup --gid $CURRENT_GID frock
RUN adduser --shell /bin/bash -G frock -D -u $CURRENT_UID frock
RUN echo "grpc.enable_fork_support = 1" >> /etc/php83/php.ini

USER frock

FROM base as local
USER root
RUN apk add --no-cache php83-pecl-xdebug
RUN chmod +x /usr/local/bin/composer

COPY 10-xdebug.ini /etc/php83/conf.d/10-xdebug.ini
USER frock