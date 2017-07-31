FROM mariadb:10.1

# File permissions
ARG PARENT_USER_ID
RUN usermod -u $PARENT_USER_ID mysql

#Log file
RUN touch /var/log/mysql/mariadb.log; \
    chmod 775 /var/log/mysql/mariadb.log; \
    chown mysql:mysql /var/log/mysql/mariadb.log; \