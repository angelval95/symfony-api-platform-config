FROM php:8.1.8-fpm-alpine AS app_php_api

ARG UID
EXPOSE $UID

RUN adduser -u ${UID} --disabled-password --gecos "" appuser
RUN mkdir /home/appuser/.ssh
RUN chown -R appuser:appuser /home/appuser/
RUN echo "StrictHostKeyChecking no" >> /home/appuser/.ssh/config
RUN echo "export COLUMNS=300" >> /home/appuser/.bashrc
RUN echo "alias sf=/appdata/www/bin/console" >> /home/appuser/.bashrc

COPY ./php.ini /usr/local/etc/php/php.ini
RUN apk add --update linux-headers
RUN docker-php-source extract \
    && apk add --update --virtual .build-deps autoconf g++ make pcre-dev icu-dev openssl-dev libxml2-dev libmcrypt-dev git libpng-dev \
# Install mysql goodness
    && docker-php-ext-install mysqli pdo pdo_mysql \
    && docker-php-ext-enable pdo_mysql \
# Instaling pecl modules
	&& pecl install apcu xdebug \
# Enable pecl modules
    && docker-php-ext-enable apcu opcache \
# Installing intl
    && apk add icu-libs icu \
    && docker-php-ext-install intl \
# Post run
	&& runDeps="$( \
		scanelf --needed --nobanner --format '%n#p' --recursive /usr/local/lib/php/extensions \
			| tr ',' '\n' \
			| sort -u \
			| awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
	)" \
	&& apk add --no-cache --virtual .app-phpexts-rundeps $runDeps \
	&& pecl clear-cache \
    && docker-php-source delete \
    && apk del --purge .build-deps \
    && rm -rf /tmp/pear \
    && rm -rf /var/cache/apk/*

#amqp extension
RUN apk --update --no-cache add build-base \
        autoconf \
        rabbitmq-c-dev

RUN pecl install amqp
RUN docker-php-ext-enable amqp


#zip extension
RUN apk add --no-cache zip libzip-dev
RUN docker-php-ext-install zip
RUN docker-php-ext-enable zip

# Install GD
RUN apk add --no-cache freetype-dev libjpeg-turbo-dev libpng-dev zlib-dev
RUN docker-php-ext-configure gd --enable-gd --with-freetype --with-jpeg
RUN docker-php-ext-install gd

RUN curl --insecure https://getcomposer.org/composer.phar -o /usr/bin/composer && chmod +x /usr/bin/composer
RUN composer self-update

RUN wget https://cs.symfony.com/download/php-cs-fixer-v2.phar -O php-cs-fixer
RUN chmod a+x php-cs-fixer
RUN mv php-cs-fixer /usr/local/bin/php-cs-fixer

RUN mkdir -p /appdata/www

WORKDIR /appdata/www