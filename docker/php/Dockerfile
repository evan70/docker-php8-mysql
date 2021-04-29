FROM php:8.0-apache

# apache2 config
ENV APACHE_DOCUMENT_ROOT /var/www/html/web
RUN sed -ri -e "s!/var/www/html!${APACHE_DOCUMENT_ROOT}!g" /etc/apache2/sites-available/*.conf \
  && sed -ri -e "s!/var/www/!${APACHE_DOCUMENT_ROOT}!g" /etc/apache2/apache2.conf /etc/apache2/conf-available/*.conf \
  && sed -i -e "s/AllowOverride None/AllowOverride All/g" /etc/apache2/apache2.conf \
  && a2enmod rewrite \
  && a2enmod headers

# php config
RUN apt update \
  && apt install -y libicu-dev autoconf make g++ gcc apt-utils git zip unzip wget \
  && apt install -y libfreetype6-dev libjpeg62-turbo-dev libpng-dev libonig-dev \
  && docker-php-ext-install -j$(nproc) pdo_mysql mysqli gd iconv intl opcache \
  && pecl install xdebug \
  && docker-php-ext-enable xdebug opcache \
  && apt upgrade -y \
  && apt autoremove -y \
  && apt clean -y
RUN { \
  echo 'short_open_tag = On'; \
  echo 'fastcgi.logging = 1'; \
  echo 'opcache.enable=1'; \
  echo 'opcache.optimization_level=0x7FFFBBFF' ; \
  echo 'opcache.revalidate_freq=0'; \
  echo 'opcache.validate_timestamps=1'; \
  echo 'opcache.memory_consumption=128'; \
  echo 'opcache.interned_strings_buffer=8'; \
  echo 'opcache.max_accelerated_files=4000'; \
  echo 'opcache.revalidate_freq=60'; \
  echo 'opcache.fast_shutdown=1'; \
  echo 'xdebug.mode=develop,debug'; \
  echo 'xdebug.client_host = host.docker.internal'; \
} > /usr/local/etc/php/conf.d/overrides.ini
# mailhog
RUN curl -sSLO https://github.com/mailhog/mhsendmail/releases/download/v0.2.0/mhsendmail_linux_amd64 \
  && chmod +x mhsendmail_linux_amd64 \
  && mv mhsendmail_linux_amd64 /usr/local/bin/mhsendmail \
  && echo 'sendmail_path = "/usr/local/bin/mhsendmail --smtp-addr=mailhog:1025"' > /usr/local/etc/php/conf.d/sendmail.ini
# https
RUN openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/ssl-cert-snakeoil.key -out /etc/ssl/certs/ssl-cert-snakeoil.pem -subj "/C=AT/ST=Vienna/L=Vienna/O=Security/OU=Development/CN=localhost" \
  && a2ensite default-ssl \
  && a2enmod ssl

# composer
COPY --from=composer /usr/bin/composer /usr/bin/composer

# permission
RUN usermod -u 1000 www-data \
  && groupmod -g 1000 www-data
