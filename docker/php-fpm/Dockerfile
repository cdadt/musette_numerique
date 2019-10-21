FROM php:7.2.23-fpm-alpine3.9

ARG BUILD_DATE

LABEL maintainer="TSEF ROGER Xavier <xavier.roger@intradef.gouv.fr>" \
        org.label-schema.build-date=$BUILD_DATE \
        php.version=${PHP_VERSION} \
        vendor="CDADT" \
        version="2.0"

ENV WORKDIR "/usr/local/apache2/htdocs/"
ENV PHPREDIS_VERSION="4.3.0"
ENV PHP_XDEBUG_VERSION 2.7.0
ENV COMPOSER_VERSION 1.8.4
ENV TIMEZONE Europe/Paris


# Installation du bash 3.1.6
RUN apk add bash

RUN apk add --update tzdata  \
    && cp /usr/share/zoneinfo/${TIMEZONE} /etc/localtime  \
    && echo "${TIMEZONE}" > /etc/timezone  \
    && apk del tzdata


# mysqli, pdo, pdo_mysql, pdo_pgsql
RUN apk add --update --no-cache libintl icu icu-dev libxml2-dev ca-certificates\
&& docker-php-ext-install mysqli pdo pdo_mysql zip soap

RUN apk add --update --no-cache --virtual .ext-deps \
    libwebp-dev \
    autoconf \
    g++ \
    make

# gd, iconv
RUN apk add --update --no-cache \
    freetype-dev \
    libjpeg-turbo-dev \
    libmcrypt-dev \
    libpng-dev \
    curl-dev\
    libxml2-dev \
    && docker-php-ext-install -j"$(getconf _NPROCESSORS_ONLN)" iconv simplexml xml curl xmlrpc\
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ --with-png-dir=/usr/include \
    && docker-php-ext-install -j"$(getconf _NPROCESSORS_ONLN)" gd
# gmp
RUN apk add gmp gmp-dev \
    && docker-php-ext-install gmp

# apcu
RUN docker-php-source extract \
    && apk add --virtual .phpize-deps-configure $PHPIZE_DEPS \
    && pecl install apcu \
    && docker-php-ext-enable apcu \
    && apk del .phpize-deps-configure \
    && docker-php-source delete

# imagick
RUN apk add autoconf g++ imagemagick-dev pcre-dev libtool make \
    && pecl install imagick \
    && docker-php-ext-enable imagick \
    && apk del autoconf g++ libtool make pcre-dev

# ssh2
RUN apk add autoconf g++ libtool make pcre-dev libssh2 libssh2-dev \
    && pecl install ssh2-1 \
    && docker-php-ext-enable ssh2 \
    && apk del autoconf g++ libtool make pcre-dev

RUN apk --no-cache add --virtual mailhog-build-dependencies \
    go \
    git \
    musl-dev
# Installation de go mhsendmail
RUN mkdir -p /usr/src/gocode && \
    export GOPATH=/usr/src/gocode && \
    go get github.com/mailhog/mhsendmail && \
    mv /usr/src/gocode/bin/mhsendmail /usr/local/bin && \
    rm -rf /usr/src/gocode && \
    apk del --purge mailhog-build-dependencies

RUN echo 'sendmail_path = /usr/local/bin/mhsendmail' > /usr/local/etc/php/php.ini

# set recommended opcache PHP.ini settings
# see https://secure.php.net/manual/en/opcache.installation.php
RUN { \
	echo 'opcache.memory_consumption=128'; \
	echo 'opcache.interned_strings_buffer=8'; \
	echo 'opcache.max_accelerated_files=4000'; \
	echo 'opcache.revalidate_freq=60'; \
	echo 'opcache.fast_shutdown=1'; \
	echo 'opcache.enable_cli=1'; \
} > /usr/local/etc/php/conf.d/opcache-recommended.ini

# set recommendé apcu PHP.ini settings
# Regarder https://secure.php.net/manual/en/apcu.configuration.php
RUN { \
        echo 'apc.shm_segments=1'; \
        echo 'apc.shm_size=256M'; \
        echo 'apc.num_files_hint=7000'; \
        echo 'apc.user_entries_hint=4096'; \
        echo 'apc.ttl=7200'; \
        echo 'apc.user_ttl=7200'; \
        echo 'apc.gc_ttl=3600'; \
        echo 'apc.max_file_size=1M'; \
        echo 'apc.stat=1'; \
} > /usr/local/etc/php/conf.d/apcu-recommended.ini

RUN docker-php-ext-configure opcache --enable-opcache && docker-php-ext-install opcache


RUN sed -i -e 's/listen.*/listen = 0.0.0.0:9000/' /usr/local/etc/php-fpm.conf

RUN echo "expose_php=0" > /usr/local/etc/php/php.ini

RUN apk add libressl2.7-libcrypto
#RUN echo "http://dl-4.alpinelinux.org/alpine/edge/community/" >> /etc/apk/repositories && apk update && \
 RUN apk upgrade && \
   apk add --update \
   libzip-dev\
   php7-session \
   php7-mcrypt \
   php7-phar \
   php7-posix \
   php7-simplexml \
   php7-fileinfo \
   php7-session \
   php7-soap \
   php7-tokenizer \
   php7-xml \
   php7-xmlreader \
   php7-json \
   php7-mbstring \
   php7-xmlwriter \
   php7-ldap

RUN apk upgrade --update && apk add \
   coreutils \
   freetype-dev \
   libjpeg-turbo-dev \
   libltdl \
   krb5-dev \
   openldap-dev \
   openldap-clients \
   libldap \
   libmcrypt-dev \
   libpng-dev \
   && docker-php-ext-configure ldap \
   && docker-php-ext-install ldap

RUN apk add --update --no-cache \
          yaml && \
      # Build dependancy for YAML \
      apk add --update --no-cache --virtual .docker-php-symfony-dependancies \
          yaml-dev && \
      pecl install yaml && \
      docker-php-ext-enable yaml;


RUN docker-php-ext-install intl
COPY ./config/php.ini /usr/local/etc/php/php.ini
RUN apk add ca-certificates
RUN mkdir /usr/share/ca-certificates/extra
# Nettoyage
RUN rm -rf /tmp/* /var/cache/apk/*

ENV LANG fr_FR.UTF-8
RUN mkdir -p /usr/local/apache2/moodledata
RUN chown www-data:www-data -R /usr/local/apache2/moodledata
RUN chmod 750 /usr/local/apache2/moodledata
RUN mkdir -p ${WORKDIR}
RUN chown www-data:www-data -R ${WORKDIR}
WORKDIR ${WORKDIR}
EXPOSE 9000

CMD ["php-fpm"]
