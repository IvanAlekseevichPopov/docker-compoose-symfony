FROM debian:jessie

MAINTAINER Ivan Popov <ivan.alekseevich.popov@gmail.com>

# File permissions
ARG PARENT_USER_ID
RUN usermod -u $PARENT_USER_ID www-data

RUN apt-get update && apt-get install -y \
    nginx

ADD nginx.conf /etc/nginx/
ADD symfony.conf /etc/nginx/sites-available/

RUN ln -s /etc/nginx/sites-available/symfony.conf /etc/nginx/sites-enabled/symfony; \
    rm  -f /etc/nginx/sites-enabled/default; \
    echo "upstream php-upstream { server php:9000; }" > /etc/nginx/conf.d/upstream.conf

CMD ["nginx"]

EXPOSE 80
EXPOSE 443
