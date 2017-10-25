FROM ubuntu:xenial

MAINTAINER Ivan Popov <ivan.alekseevich.popov@gmail.com>

# install php php-fpm php extentions
RUN apt-get update; \
    apt-get install -y software-properties-common libmagickwand-dev imagemagick git curl unzip wget libgearman-dev; \
    add-apt-repository -y ppa:ondrej/php || echo 1; \
    apt-get update; \
    apt-get install -y --allow-unauthenticated php7.1 php7.1-fpm php7.1-zip php7.1-mysql php7.1-curl php7.1-opcache php7.1-xdebug php-redis php-imagick php-dev php7.1-mbstring php7.1-sqlite3

# Gearman plugin install
RUN cd /tmp/; \
    wget https://github.com/wcgallego/pecl-gearman/archive/master.zip; \
    unzip master.zip; \
    cd pecl-gearman-master; \
    phpize; \
    ./configure; \
    make; \
    make install; \
    echo "extension=gearman.so" > /etc/php/7.1/mods-available/gearman.ini; \
    phpenmod -v ALL -s ALL gearman

# Rights on pid and socket
RUN mkdir -p /run/php; \
    chown -R www-data:www-data /run/php; \
    chmod -R 775 /run/php;

# Config forward
ADD php.ini /etc/php/7.1/fpm/php.ini
ADD xdebug.ini /etc/php/7.1/mods-available/xdebug.ini
ADD fpm.conf /etc/php/7.1/fpm/pool.d/www.conf

# Install Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Set timezone
RUN rm /etc/localtime; \
    ln -s /usr/share/zoneinfo/Europe/Moscow /etc/localtime

# File permissions
ARG PARENT_USER_ID
RUN usermod -u $PARENT_USER_ID www-data

WORKDIR /var/www/symfony

RUN mkdir /var/www/history

ENV HISTFILE '/var/www/history/bash_history_php'
ENV PROMPT_COMMAND 'history -a'
ENV HISTTIMEFORMAT '%h %d %H:%M:%S '
ENV HISTFILESIZE 20000

USER www-data

CMD ["php-fpm7.1", "-F"]
EXPOSE 9000