FROM php:8.3.11-fpm

# Update package list and install dependencies
RUN apt-get update && apt-get install -y \
    libzip-dev \
    libpng-dev \
    postgresql-client \
    libpq-dev \
    nodejs \
    npm \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Install Composer
COPY --from=composer:latest /usr/bin/composer /usr/local/bin/composer

ENV COMPOSER_ALLOW_SUPERUSER=1

#RUN docker-php-ext-install pdo gd bcmath zip mysqli mysqlnd pdo_mysql \
    #&& pecl install redis \
    #&& docker-php-ext-enable redis

# Install required packages
ADD https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions /usr/local/bin/

RUN chmod +x /usr/local/bin/install-php-extensions; \
    install-php-extensions mysqli mysqlnd pdo pdo_mysql zip

WORKDIR /usr/share/nginx/html/

# Copy the codebase
COPY . ./

# Run composer install for production and give permissions
RUN sed 's_@php artisan package:discover_/bin/true_;' -i composer.json \
    && composer install --ignore-platform-req=php --no-dev --optimize-autoloader \
    && composer clear-cache \
    && php artisan package:discover --ansi \
    && chmod -R 775 storage \
    && chown -R www-data:www-data storage \
    && mkdir -p storage/framework/sessions storage/framework/views storage/framework/cache

# Copy entrypoint
COPY ./scripts/php-fpm-entrypoint /usr/local/bin/php-entrypoint

# Give permisisons to everything in bin/
RUN chmod a+x /usr/local/bin/*

ENTRYPOINT ["/usr/local/bin/php-entrypoint"]

CMD ["php-fpm"]

