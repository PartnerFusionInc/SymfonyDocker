FROM php:7.0-apache

EXPOSE 80
EXPOSE 443

RUN useradd -m webuser
RUN usermod -aG www-data webuser

# Apache Config files
ADD ./config/php/php.ini /usr/local/etc/php/
ADD ./config/apache2/mods-available/* /etc/apache2/mods-available/

RUN apt-get update && apt-get install -my wget gnupg

# Install NodeJS 8.x
RUN curl -sL https://deb.nodesource.com/setup_8.x | bash -

# Install memcache extension
RUN set -x \
    && apt-get update && apt-get install -y --no-install-recommends unzip libssl-dev libpcre3 libpcre3-dev libmemcached-dev git build-essential ssh python-setuptools python-pip groff-base libmcrypt-dev libicu-dev zlib1g-dev nodejs

# Installing AWS-cli
RUN pip install awscli

# Composer installation
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Install memcached extension
RUN git clone https://github.com/php-memcached-dev/php-memcached /usr/src/php/ext/memcached \
  && cd /usr/src/php/ext/memcached && git checkout -b php7 origin/php7 \
  && docker-php-ext-configure memcached \
  && docker-php-ext-install memcached

# PHP libs
RUN docker-php-ext-configure intl \
    && docker-php-ext-install opcache zip intl -j$(nproc) iconv mcrypt pdo pdo_mysql sockets

# Self-signed certificate
RUN openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/ssl-cert-snakeoil.key -out /etc/ssl/certs/ssl-cert-snakeoil.pem -subj "/C=AT/ST=Vienna/L=Vienna/O=Security/OU=Development/CN=example.com"

# Installation of Opcode cache
RUN ( \
  echo "opcache.enable=1"; \  
  echo "opcache.enable_cli=0"; \  
  echo "opcache.memory_consumption=256"; \
  echo "opcache.interned_strings_buffer=8"; \
  echo "opcache.max_wasted_percentage=5"; \
  echo "opcache.use_cwd=1"; \
  echo "opcache.revalidate_freq=0"; \
  echo "opcache.validate_timestamps=0"; \
  echo "opcache.max_accelerated_files=20000"; \
  echo "opcache.fast_shutdown=1"; \  
  echo "opcache.save_comments=1"; \  
  echo "opcache.realpath_cache_size=4096K"; \  
  echo "opcache.realpath_cache_ttl=600"; \  
  echo "opcache.load_comments=0"; \  
  echo "opcache.revalidate_path=1"; \  
  echo "opcache.enable_file_override=1"; \  
  echo "opcache.max_file_size=2097152"; \
  ) > /usr/local/etc/php/conf.d/opcache-recommended.ini

# Apache modules
RUN a2enmod rewrite
RUN a2enmod ssl
RUN a2enmod headers
RUN a2enmod status
RUN a2enmod deflate
RUN a2dismod 'mpm_*' && a2enmod mpm_prefork
