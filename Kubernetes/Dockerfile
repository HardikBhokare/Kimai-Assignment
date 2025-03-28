FROM php:8.1-apache

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    git \
    zip \
    unzip \
    libzip-dev \
    libpng-dev \
    libjpeg-dev \
    libwebp-dev \
    libfreetype6-dev \
    libicu-dev \
    libonig-dev \
    libxml2-dev \
    && rm -rf /var/lib/apt/lists/*

RUN docker-php-ext-install -j$(nproc) \
    mysqli \
    pdo_mysql \
    zip \
    gd \
    intl \
    exif \
    opcache \
    xml \
    && docker-php-ext-configure gd \
    --with-freetype \
    --with-jpeg \
    --with-webp \
    && docker-php-ext-enable gd intl


RUN a2enmod rewrite headers

COPY . /var/www/html

WORKDIR /var/www/html

RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer && \
    composer install --no-interaction --optimize-autoloader --no-dev

RUN chown -R www-data:www-data /var/www/html/var

CMD ["apache2-foreground"]
