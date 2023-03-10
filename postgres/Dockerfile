FROM postgres:14.5

ENV TZ 'Europe/Moscow'
    RUN echo $TZ > /etc/timezone && \
    apt-get update && apt-get install -y tzdata

RUN apt-get update \
    && apt-get -y install \
        postgresql-14-repack

# - Set the locale
RUN sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen && \
    sed -i -e 's/# ru_RU.UTF-8 UTF-8/ru_RU.UTF-8 UTF-8/' /etc/locale.gen && \
    echo 'LANG="ru_RU.UTF-8"'>/etc/default/locale && \
    dpkg-reconfigure --frontend=noninteractive locales && \
    update-locale LANG=ru_RU.UTF-8

ENV LANG ru_RU.UTF-8
ENV LANGUAGE ru_RU.UTF-8
ENV LC_ALL ru_RU.UTF-8

CMD ["postgres"]

EXPOSE 5432
