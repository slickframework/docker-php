FROM php:7.1-fpm
MAINTAINER Filipe Silva <silvam.filipe@gmail.com>

COPY bin/* /usr/local/bin/
RUN mkdir -p /usr/local/etc/php/disabled

# Install useful extensions
RUN apt-install \
      libfreetype6-dev \
      libjpeg62-turbo-dev \
      libmcrypt-dev \
      libpng12-dev \
      libicu-dev \
      libmemcached-dev \
  && docker-php-ext-install \
      mcrypt \
      mysqli \
      pdo_mysql \
      mbstring \
  && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
  && docker-php-ext-install gd

RUN apt-install zlib1g-dev libicu-dev g++ && \
    docker-php-ext-configure intl && \
    docker-php-ext-install intl && \
    apt-purge g++

ENV MEMCACHED_VERSION 3.0.3
RUN { yes '/usr' | pecl install memcached-$MEMCACHED_VERSION; } \
    && echo "extension=memcached.so" >> /usr/local/etc/php/conf.d/memcached.ini

ENV USE_XDEBUG no
ENV XDEBUG_VERSION 2.5.4
RUN docker-php-pecl-install xdebug-$XDEBUG_VERSION && \
    echo "xdebug.remote_enable=on\nxdebug.remote_connect_back=on" > /usr/local/etc/php/conf.d/xdebug.ini && \
    mkdir -p /usr/local/etc/php/xdebug.d && \
    mv /usr/local/etc/php/conf.d/*xdebug.ini /usr/local/etc/php/xdebug.d/


# Use SSMTP
RUN apt-install ssmtp && \
    groupadd -g 500 ssmtp && \
    chown :ssmtp /usr/sbin/ssmtp && \
    chmod g+s /usr/sbin/ssmtp && \
    echo "sendmail_path = /usr/sbin/ssmtp -t" > /usr/local/etc/php/conf.d/sendmail.ini && \
    mv /etc/ssmtp/ssmtp.conf /etc/ssmtp/ssmtp.conf~


# Grab gosu for easy step-down from root
ENV GOSU_VERSION 1.10
RUN set -x \
  && curl -sSLo /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture)" \
  && curl -sSLo /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture).asc" \
  && export GNUPGHOME="$(mktemp -d)" \
  && gpg --keyserver ha.pool.sks-keyservers.net --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4 \
  && gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu \
  && rm -r "$GNUPGHOME" /usr/local/bin/gosu.asc \
  && chmod +x /usr/local/bin/gosu \
  && gosu nobody true


# Include composer
RUN apt-install git rsync zlib1g-dev && \
    docker-php-ext-install zip

ENV COMPOSER_HOME /var/www/.composer
ENV COMPOSER_VERSION 1.4.2
ENV PATH vendor/bin:$COMPOSER_HOME/vendor/bin:$PATH

RUN curl -sS https://getcomposer.org/installer | php -- \
      --install-dir=/usr/local/bin \
      --filename=composer \
      --version=${COMPOSER_VERSION}

RUN mkdir -p $COMPOSER_HOME/cache && \
    chown -R www-data:www-data /var/www && \
    echo "phar.readonly = off" > /usr/local/etc/php/conf.d/phar.ini
VOLUME $COMPOSER_HOME/cache


# Add configs
COPY etc/*.ini /usr/local/etc/php/
RUN cd /usr/local/etc/php && cp prod.ini conf.d/environment.ini
RUN sed -i -e '/^access.log/ s/^/;/' /usr/local/etc/php-fpm.d/docker.conf && \
    sed -i -e "s/^pm = dynamic/pm = ondemand/" /usr/local/etc/php-fpm.d/www.conf


# Add entrypoint
COPY init.d /docker-entrypoint-init.d/
COPY docker-entrypoint.sh /
ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["php-fpm"]
WORKDIR /usr/src/app
