# See https://github.com/docker-library/php/blob/4677ca134fe48d20c820a19becb99198824d78e3/7.0/fpm/Dockerfile
FROM php:7.1-fpm

MAINTAINER Ivan Popov <ivan.alekseevich.popov@gmail.com>

# File permissions
ARG PARENT_USER_ID
RUN usermod -u $PARENT_USER_ID www-data

# Install Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Set timezone
RUN rm /etc/localtime; \
    ln -s /usr/share/zoneinfo/Europe/Moscow /etc/localtime

# Type docker-php-ext-install to see available extensions
RUN docker-php-ext-install pdo pdo_mysql mbstring

# install xdebug redis extentions
RUN pecl install xdebug redis; \
    docker-php-ext-enable xdebug; \
    docker-php-ext-enable redis

RUN echo 'alias sf="php app/console"' >> ~/.bashrc; \
    echo 'alias sf3="php bin/console"' >> ~/.bashrc

WORKDIR /var/www/symfony

USER www-data