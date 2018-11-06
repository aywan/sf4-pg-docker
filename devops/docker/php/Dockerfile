##
## Production build.
##
FROM php:7.2.11-fpm-alpine3.7 as prod

ARG XHPROF_VER=v2.0.2
ARG MEMCACHED_VER=3.0.4

RUN : "------ user works --------" \
        && deluser xfs \
        && deluser www-data \
        && addgroup -g 33 www-data \
        && adduser -G www-data -u 33 -g "php-fpm user" -s /bin/nologin -D www-data \
    \
    && : "------ lib dependency ----" \
    && apk add --no-cache --update \
      icu-dev \
      libpq \
      libmemcached \
      postgresql-dev \
      libpng-dev \
      bash \
    && docker-php-source extract \
    && apk add --no-cache --update --virtual .phpize-deps $PHPIZE_DEPS \
    && apk add --no-cache --update --virtual building-ext-deps \
      git \
      zlib-dev \
      libmemcached-dev \
    && docker-php-ext-install \
      gd \
      pdo_pgsql \
      pdo_mysql \
      intl \
      bcmath \
      zip \
      pcntl \
    \
   && pecl install memcached-${MEMCACHED_VER} \
    \ && docker-php-ext-enable memcached \
    \
   && pecl install timezonedb \
    \ && docker-php-ext-enable timezonedb \
    \
    && git clone --depth=1 https://github.com/longxinH/xhprof /usr/src/xhprof \
      && cd /usr/src/xhprof/extension \
      && phpize \
      && ./configure \
      && make \
      && make install \
      && cd / \
      && rm -rf /usr/src/xhprof || true \
    \
    && apk del building-ext-deps \
    && apk del .phpize-deps \
    && docker-php-source delete \
    && ln -s /usr/local/etc/php /etc/php

# COPY config/stand/ini/ $PHP_INI_DIR/
COPY utils/suwd.sh /usr/bin/suwd

#COPY ssh-keys /root/.ssh/
#RUN chown root:root /root/.ssh \
 #   && chmod 600 /root/.ssh/* \
  #  && apk --no-cache add openssh-client \
  #  && ssh -o "BatchMode=yes" -o "StrictHostKeyChecking=no" git@gitlab.dev.magdv.com

WORKDIR /code

### end prod build

##
## Developers build.
##
FROM prod as dev

ARG XDEBUG_VER=2.6.1

RUN docker-php-source extract \
    && apk add --no-cache --update --virtual .phpize-deps $PHPIZE_DEPS \
    \
    && pecl install xdebug-${XDEBUG_VER} \
    && docker-php-ext-enable xdebug \
    \
    && apk del .phpize-deps \
    && docker-php-source delete \
    && : "----- composer -----" \
      && php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
      && php -r "if (hash_file('SHA384', 'composer-setup.php') === '93b54496392c062774670ac18b134c3b3a95e5a5e5c8f1a9f115f203b75bf9a129d5daa8ba6a13e2cc8a1da0806388a8') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;" \
      && php composer-setup.php \
      && php -r "unlink('composer-setup.php');" \
      && mv ./composer.phar /composer.phar \
      && ln -s /composer.phar /usr/bin/composer \
    && apk add --no-cache --update \
      mc \
      git

### end dev build