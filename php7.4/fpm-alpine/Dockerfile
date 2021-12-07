ARG PHP_VERSION=7.4.26
ARG PHP_ALPINE_VERSION=3.14
ARG PHP_EXTENSION_INSTALLER_VERSION=1.4.6
ARG COMPOSER_VERSION=2.1.14
ARG SNIPEIT_RELEASE=5.3.3


# cannot use arguments with 'COPY --from' workaround
# https://github.com/moby/moby/issues/34482#issuecomment-454716952
# 
# install php extensions inside docker containers easily
# https://github.com/mlocati/docker-php-extension-installer
FROM mlocati/php-extension-installer:${PHP_EXTENSION_INSTALLER_VERSION} as php-extension-installer
FROM composer:${COMPOSER_VERSION} AS composer

# Final stage
FROM php:${PHP_VERSION}-fpm-alpine${PHP_ALPINE_VERSION} AS source
LABEL maintainer="Mateus Villar <mromeravillar@gmail.com>"

ENV ENVIRONMENT production
ARG PACKAGES="\
		mysql-client \
"
ARG SNIPEIT_RELEASE
ENV SNIPEIT_RELEASE ${SNIPEIT_RELEASE}

# cribbed from wordpress-fpm-alpine image
#
# set recommended PHP.ini settings
# see https://secure.php.net/manual/en/opcache.installation.php
RUN set -eux; \
	docker-php-ext-enable opcache; \
	{ \
		echo 'opcache.memory_consumption=128'; \
		echo 'opcache.interned_strings_buffer=8'; \
		echo 'opcache.max_accelerated_files=4000'; \
		echo 'opcache.revalidate_freq=2'; \
		echo 'opcache.fast_shutdown=1'; \
	} > /usr/local/etc/php/conf.d/opcache-recommended.ini
# https://wordpress.org/support/article/editing-wp-config-php/#configure-error-logging
RUN { \
# https://www.php.net/manual/en/errorfunc.constants.php
# https://github.com/docker-library/wordpress/issues/420#issuecomment-517839670
		echo 'error_reporting = E_ERROR | E_WARNING | E_PARSE | E_CORE_ERROR | E_CORE_WARNING | E_COMPILE_ERROR | E_COMPILE_WARNING | E_RECOVERABLE_ERROR'; \
		echo 'display_errors = Off'; \
		echo 'display_startup_errors = Off'; \
		echo 'log_errors = On'; \
		echo 'error_log = /dev/stderr'; \
		echo 'log_errors_max_len = 1024'; \
		echo 'ignore_repeated_errors = On'; \
		echo 'ignore_repeated_source = Off'; \
		echo 'html_errors = Off'; \
	} > /usr/local/etc/php/conf.d/error-logging.ini

# install php extensions
COPY --from=php-extension-installer /usr/bin/install-php-extensions /usr/local/bin/
RUN set -eux; \
    install-php-extensions \
        bcmath \
        gd \
        ldap \
        mysqli \
        pdo_mysql \
        zip; \
    rm -f /usr/local/bin/install-php-extensions; \
# install prerequisites packages
    apk add --no-cache \
        ${PACKAGES};

COPY --from=composer /usr/bin/composer /usr/local/bin
ARG COMPOSER_ALLOW_SUPERUSER=1
RUN set -eux; \
# download and extract snipeit tarball
	curl -o snipeit.tar.gz -fL "https://github.com/snipe/snipe-it/archive/v$SNIPEIT_RELEASE.tar.gz"; \
	tar -xzf snipeit.tar.gz --strip-components=1 -C /var/www/html/; \
	rm snipeit.tar.gz; \
# install composer php dependencies
    composer update \
        --no-cache \
        --no-dev \
        --optimize-autoloader \
        --working-dir=/var/www/html; \
	rm -f /usr/local/bin/composer; \
	chown -R www-data:www-data /var/www/html;

VOLUME [ "/var/lib/snipeit" ]

COPY --chmod=655 docker-snipeit-entrypoint /usr/local/bin/

ENTRYPOINT [ "docker-snipeit-entrypoint" ]
CMD [ "docker-php-entrypoint", "php-fpm" ]