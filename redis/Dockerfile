FROM redis:4

# File permissions
ARG PARENT_USER_ID
RUN usermod -u $PARENT_USER_ID redis

#Log file
RUN touch /var/log/redis.log; \
    chmod 775 /var/log/redis.log; \
    chown redis:redis /var/log/redis.log

#Redis start
CMD ["redis-server", "/usr/local/etc/redis/redis.conf"]