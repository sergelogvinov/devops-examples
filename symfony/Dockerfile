# syntax = docker/dockerfile:1.10
########################################

FROM composer:2.8.1 AS composer

########################################

FROM php:8.3-fpm-alpine3.20 AS base

# Install dependencies
RUN apk add --no-cache tzdata nginx curl \
    bash vim zip unzip icu-libs \
    c-client make ca-certificates imap gmp gettext libssh2 yaml \
    libintl libxslt libpng libwebp libjpeg-turbo freetype imap \
    oniguruma libxslt libpq vips \
    gmp libzip libxml2 freetds

# Install php dependencies
RUN apk add --no-cache --virtual .build_deps \
    libpng-dev libwebp-dev libjpeg-turbo-dev freetype-dev imap-dev \
    linux-headers oniguruma-dev libxslt-dev postgresql-dev vips-dev \
    libssh2-dev gmp-dev libzip-dev libxml2-dev freetds-dev yaml-dev \
    $PHPIZE_DEPS && \
    docker-php-ext-install \
      pdo_pgsql \
      pgsql\
      bcmath \
      mbstring \
      xml \
      gd \
      exif \
      zip \
      soap \
      intl \
      xsl \
      pcntl \
      sockets \
      sysvmsg \
      sysvsem \
      sysvshm \
      opcache \
      imap \
      gmp && \
    pecl install -o -f yaml && docker-php-ext-enable yaml && \
    # Clean up
    rm -rf /tmp/pear && docker-php-source delete && \
    apk del --purge .build_deps && rm -rf /var/cache/apk/*

# Install composer
COPY --from=composer /usr/bin/composer /usr/local/bin/composer
COPY config/php.ini ${PHP_INI_DIR}/conf.d/99-php.ini
COPY config/nginx.conf /etc/nginx/http.d/default.conf

USER www-data

ENV WORKDIR=/var/symfony
WORKDIR ${WORKDIR}

COPY --chown=www-data:www-data ["composer.json", "composer.lock", "symfony.lock", "${WORKDIR}"]
COPY --chown=www-data:www-data ["bin/", "${WORKDIR}/bin/"]
RUN composer install --no-interaction --no-scripts --optimize-autoloader

COPY --chown=www-data:www-data [".", "${WORKDIR}"]

ARG APP_ENV=dev
ENV APP_ENV=${APP_ENV}

EXPOSE 9000

CMD ["/usr/local/sbin/php-fpm", "-F"]
