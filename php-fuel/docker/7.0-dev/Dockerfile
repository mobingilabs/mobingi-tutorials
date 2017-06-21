FROM ubuntu:16.04

LABEL maintainer="mobingi,Inc."

RUN apt-get update && apt-get install -y --no-install-recommends \
		apache2 \
		software-properties-common \
		supervisor \
		vim \
		curl \
		libcurl4-gnutls-dev \
		libexpat1-dev \
		gettext \
		libz-dev \
		libssl-dev \
		git \
	&& apt-get clean \
	&& rm -fr /var/lib/apt/lists/*

RUN LC_ALL=C.UTF-8 add-apt-repository ppa:ondrej/php

RUN apt-get update && apt-get install -y --no-install-recommends \
		libapache2-mod-php7.0 \
		php7.0 \
		php7.0-cli \
		php7.0-curl \
		php7.0-dev \
		php7.0-gd \
		php7.0-imap \
		php7.0-mbstring \
		php7.0-mcrypt \
		php7.0-mysql \
		php7.0-pgsql \
		php7.0-pspell \
		php7.0-xml \
		php7.0-xmlrpc \
		php-apcu \
		php-memcached \
		php-pear \
		php-redis \
	&& apt-get clean \
	&& rm -fr /var/lib/apt/lists/*

ENV COMPOSER_HOME /composer
## Add global binary directory to PATH and make sure to re-export it
ENV PATH /composer/vendor/bin:$PATH
## Allow Composer to be run as root
ENV COMPOSER_ALLOW_SUPERUSER 1
## Setup the Composer installer
RUN curl -o /tmp/composer-setup.php https://getcomposer.org/installer \
  && php /tmp/composer-setup.php --no-ansi --install-dir=/usr/local/bin --filename=composer


RUN a2enmod rewrite
COPY conf/000-default.conf /etc/apache2/sites-available/000-default.conf

COPY conf/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

COPY script/run.sh /run.sh
RUN chmod 755 /run.sh

COPY conf/config /config


EXPOSE 80
CMD ["/run.sh"]
