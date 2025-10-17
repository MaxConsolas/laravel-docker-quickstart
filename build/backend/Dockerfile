# Multi-stage build for size optimization
FROM ghcr.io/step3003/php-fpm-alpine:4.2.0 AS base

USER root

# Copy php.ini configuration
COPY build/backend/php.ini /usr/local/etc/php/conf.d/php.ini

# Install only necessary packages
RUN apk add --no-cache \
    libxslt-dev libxml2-dev libgcc libstdc++ libx11 glib libxrender libxext libintl less libzip-dev zip \
    icu-dev git tzdata \
    g++ gcc libc-dev make re2c \
    && docker-php-ext-install zip pcntl intl soap bcmath \
    && docker-php-ext-enable soap bcmath \
    && rm -rf /var/cache/apk/*

USER php
WORKDIR /app

# Stage 1: Vendor dependencies
FROM base AS vendor

COPY --chown=php:php backend/composer.json backend/composer.lock ./
RUN composer install --no-dev --no-scripts --prefer-dist --optimize-autoloader --no-interaction

# Stage 2: Production backend
FROM base AS back

# Copy only vendor from previous stage
COPY --chown=php:php --from=vendor /app/vendor ./vendor

# Copy application code
COPY --chown=php:php backend ./

# Optimize Composer
RUN composer dump-autoload --optimize --no-dev

# Create storage folders with correct permissions
RUN mkdir -p storage/app storage/framework/cache storage/framework/sessions storage/framework/views storage/logs \
    && chown -R php:php storage \
    && chmod -R 755 storage

# Clear cache
RUN rm -rf /tmp/* /var/tmp/*

# Stage 3: Nginx (minimal)
FROM nginx:1.25-alpine AS nginx

# Remove unnecessary files
RUN rm -rf /etc/nginx/conf.d/default.conf \
    && rm -rf /usr/share/nginx/html/* \
    && rm -rf /var/cache/apk/*

# Copy configuration
COPY build/backend/nginx/nginx.conf /etc/nginx/nginx.conf
COPY build/backend/nginx/server.conf /etc/nginx/templates/nginx.conf.template

# Copy only public files
COPY --from=back /app/public /app/public

# Nginx user already exists in the image

CMD ["nginx", "-g", "daemon off;"]
