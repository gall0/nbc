# Recommended PHP Version
# https://www.drupal.org/docs/8/system-requirements/php-requirements
FROM php:7.3.1-apache

# install the PHP extensions we need
RUN set -ex; \
	\
	if command -v a2enmod; then \
		a2enmod rewrite; \
	fi; \
	\
	savedAptMark="$(apt-mark showmanual)"; \
	\
	apt-get update; \
	apt-get install -y \
		libjpeg-dev \
		libpng-dev \
		libpq-dev \
        libzip-dev \
		libfreetype6-dev \
	; \
	\
	docker-php-ext-configure gd --with-freetype-dir=/usr --enable-gd-native-ttf --with-png-dir=/usr --with-jpeg-dir=/usr; \
	docker-php-ext-install -j "$(nproc)" \
		gd \
		opcache \
		pdo_mysql \
		pdo_pgsql \
		zip \
	; \
	\
# reset apt-mark's "manual" list so that "purge --auto-remove" will remove all build dependencies
	apt-mark auto '.*' > /dev/null; \
	apt-mark manual $savedAptMark; \
	ldd "$(php -r 'echo ini_get("extension_dir");')"/*.so \
		| awk '/=>/ { print $3 }' \
		| sort -u \
		| xargs -r dpkg-query -S \
		| cut -d: -f1 \
		| sort -u \
		| xargs -rt apt-mark manual; \
	\
	apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
	rm -rf /var/lib/apt/lists/*

# set recommended PHP.ini settings
# see https://secure.php.net/manual/en/opcache.installation.php
RUN { \
		echo 'opcache.memory_consumption=128'; \
		echo 'opcache.interned_strings_buffer=8'; \
		echo 'opcache.max_accelerated_files=4000'; \
		echo 'opcache.revalidate_freq=60'; \
		echo 'opcache.fast_shutdown=1'; \
		echo 'opcache.enable_cli=1'; \
	} > /usr/local/etc/php/conf.d/opcache-recommended.ini

RUN apt-get update && apt-get install -y \
## Add additional tools needed.
		curl \
		git \
		mysql-client \
		vim \
		wget \
		iputils-ping 
## End of additional tools.

## 
## Drupal composer 
##
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" && \
	php composer-setup.php && \
	mv composer.phar /usr/local/bin/composer && \
	php -r "unlink('composer-setup.php');"

##
## Drush
##
RUN wget -O drush.phar https://github.com/drush-ops/drush-launcher/releases/download/0.4.2/drush.phar && \
	chmod +x drush.phar && \
	mv drush.phar /usr/local/bin/drush
##
## Apache configuration
##
COPY conf/apache-drupal.conf /etc/apache2/sites-enabled/000-default.conf

##
## PHP Configuration
##
COPY conf/upload.ini /usr/local/etc/php/conf.d/upload.ini
COPY conf/memory_limit.ini /usr/local/etc/php/conf.d/memory_limit.ini

##
## Set working directory
##
WORKDIR /app

# vim:set ft=dockerfile: