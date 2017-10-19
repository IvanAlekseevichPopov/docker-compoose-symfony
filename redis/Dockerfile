FROM redis:4

# File permissions
ARG PARENT_USER_ID
RUN usermod -u $PARENT_USER_ID redis

COPY entrypoint.sh /usr/local/bin/
ENTRYPOINT ["entrypoint.sh"]

EXPOSE 6379
CMD ["redis-server", "/usr/local/etc/redis/redis.conf"]