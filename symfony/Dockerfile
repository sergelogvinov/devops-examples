# syntax = docker/dockerfile:1.10
########################################

FROM composer:2.8.1 AS composer

########################################
#
# Base image
#

FROM php:8.3-fpm-alpine3.20 AS base

# Install common dependencies
#
# Nginx is required to route requests to the PHP-FPM server
# Curl for debugging purposes you can remove it
# All other packages are required to run PHP extensions
#
RUN apk add --no-cache tzdata nginx curl \
    bash vim zip unzip icu-libs \
    c-client make ca-certificates imap gmp gettext libssh2 yaml \
    libintl libxslt libpng libwebp libjpeg-turbo freetype imap \
    oniguruma libxslt libpq vips \
    gmp libzip libxml2 freetds

########################################
#
# Build the project with development dependencies
#

FROM base AS builder

# Install php dependencies to build extensions
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
#
# Composer is required to install the project dependencies
# It does not need to be installed in the final image
#
COPY --from=composer /usr/bin/composer /usr/local/bin/composer

# Use unprivileged user to build environment
USER www-data

ENV WORKDIR=/var/symfony
WORKDIR ${WORKDIR}

COPY --chown=www-data:www-data ["composer.json", "composer.lock", "symfony.lock", "${WORKDIR}"]
COPY --chown=www-data:www-data ["bin/", "${WORKDIR}/bin/"]
RUN composer install --no-interaction --no-scripts --optimize-autoloader

########################################
#
# The final release image
#

FROM base AS release

# Copy necessary files
COPY config/php.ini ${PHP_INI_DIR}/conf.d/99-php.ini
COPY config/nginx.conf /etc/nginx/http.d/default.conf

# Use unprivileged user to run the application
USER www-data

ENV WORKDIR=/var/symfony
WORKDIR ${WORKDIR}

# Copy the project files
#
# Copy the vendor directory from the builder image, it has all the dependencies
# Copy the project files from the current directory
#
COPY --chown=www-data:www-data --from=builder ["${WORKDIR}/vendor/", "${WORKDIR}/vendor/"]
COPY --chown=www-data:www-data [".", "${WORKDIR}"]

# Default environment for the symfony application
ARG APP_ENV=dev
ENV APP_ENV=${APP_ENV}

EXPOSE 9000

CMD ["/usr/local/sbin/php-fpm", "-F"]
