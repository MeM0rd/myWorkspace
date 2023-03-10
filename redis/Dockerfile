FROM redis:7.0.4

## For security settings uncomment, make the dir, copy conf, and also start with the conf, to use it
#RUN mkdir -p /usr/local/etc/redis

RUN \
    if [ ! -f /var/log/redis.log ]; then \
        touch /var/log/redis.log; \
        chown redis:redis /var/log/redis.log; \
    fi;

COPY etc/redis.conf /usr/local/etc/redis/redis.conf

VOLUME /data

EXPOSE 6379

#CMD ["redis-server", "/usr/local/etc/redis/redis.conf"]
CMD ["redis-server"]
