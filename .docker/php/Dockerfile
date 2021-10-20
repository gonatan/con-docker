FROM php:7.2-fpm

# install base
RUN apt-get update \
    && apt-get install -y \
        gnupg \
        locales \
        openssl \
        git \
        unzip \
        libzip-dev \
        libmagickwand-dev \
        imagemagick \
        libicu-dev \
        libxml2-dev \
        libwebp-dev \
        libpng-dev \
        libxpm-dev \
        libfreetype6-dev \
        graphicsmagick \
        libldap2-dev

# install php extensions
RUN docker-php-ext-install mysqli pdo_mysql intl calendar zip opcache \
    && docker-php-ext-configure gd --with-png-dir=/usr/include/ --with-jpeg-dir=/usr/include/ --with-freetype-dir=/usr/include/ \
    && docker-php-ext-install gd

# install php extensions via pecl
RUN pecl install imagick xdebug \
    && docker-php-ext-enable imagick xdebug \
    && curl -sS https://getcomposer.org/installer | php -- --version=1.10.17 --install-dir=/usr/local/bin --filename=composer \
    && pecl clear-cache \
    && docker-php-source delete

# Set the locale
RUN sed -i '/de_DE.UTF-8/s/^# //g' /etc/locale.gen \
    && locale-gen
ENV LANG de_DE.UTF-8
ENV LANGUAGE de_DE:de
ENV LC_ALL de_DE.UTF-8

# Copy xdebug and php config.
COPY conf.d/* /usr/local/etc/php/conf.d/
COPY php-fpm.d/* /usr/local/etc/php-fpm.d/

# Setup php ini config
ARG PHP_INI
RUN mv /usr/local/etc/php/${PHP_INI} /usr/local/etc/php/php.ini

# overwrite default www-data id and group from alpine image, create local user with access to our git
# use shadow to resolve linux UID_MAX limits for out active directory users
# -K MAIL_DIR=/dev/null - create user withou mail spool
ARG USER_ID
ARG GROUP_ID
RUN usermod -u 55100 www-data \
    && groupmod -g 55101 www-data \
    && if [ "$GROUP_ID" -ne "0" ]; then /usr/sbin/groupadd -g ${GROUP_ID} human; fi \
    && if [ "$GROUP_ID" -ne "0" ] && [ "$USER_ID" -ne "0" ]; then /usr/sbin/useradd -l -m -K MAIL_DIR=/dev/null -s /bin/sh -g ${GROUP_ID} -u ${USER_ID} arthurdent; fi

# Set default working directory
ARG PROJECT_WORKDIR
WORKDIR ${PROJECT_WORKDIR}
