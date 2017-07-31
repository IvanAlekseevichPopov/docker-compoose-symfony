# See https://github.com/docker-library/php/blob/4677ca134fe48d20c820a19becb99198824d78e3/7.0/fpm/Dockerfile
FROM php:7.1-fpm

MAINTAINER Ivan Popov <ivan.alekseevich.popov@gmail.com>

RUN echo $UID
RUN echo $UID
# File permissions
ARG PARENT_USER_ID
RUN usermod -u $PARENT_USER_ID www-data

# Type docker-php-ext-install to see available extensions
RUN docker-php-ext-install pdo pdo_mysql

# install xdebug redis extentions
RUN apt-get update; \
    apt-get install -y libmagickwand-dev imagemagick; \
    pecl install xdebug redis imagick; \
    docker-php-ext-enable xdebug; \
    docker-php-ext-enable redis

ADD xdebug.ini /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "extension=imagick.so" > /usr/local/etc/php/conf.d/docker-php-ext-imagic.ini

# Install Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Set timezone
RUN rm /etc/localtime; \
    ln -s /usr/share/zoneinfo/Europe/Moscow /etc/localtime

WORKDIR /var/www/symfony

#USER www-data