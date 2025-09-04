FROM php:8.4-apache

# ติดตั้งเครื่องมือ build สำหรับคอมไพล์ PHP extensions (ใช้ชุด $PHPIZE_DEPS)
RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
      $PHPIZE_DEPS libpq-dev \
    ; \
    rm -rf /var/lib/apt/lists/*

# เปิด mod_rewrite และปรับ vhost ให้ AllowOverride All (ใช้ .htaccess ได้)
RUN set -eux; \
    a2enmod rewrite; \
    { \
      echo '<VirtualHost *:80>'; \
      echo '  DocumentRoot /var/www/html'; \
      echo '  <Directory /var/www/html>'; \
      echo '    AllowOverride All'; \
      echo '    Require all granted'; \
      echo '  </Directory>'; \
      echo '  ErrorLog ${APACHE_LOG_DIR}/error.log'; \
      echo '  CustomLog ${APACHE_LOG_DIR}/access.log combined'; \
      echo '</VirtualHost>'; \
    } > /etc/apache2/sites-available/000-default.conf

# ติดตั้ง PHP extensions: PDO, PDO MySQL, OPcache
RUN docker-php-ext-install -j"$(nproc)" pdo pdo_pgsql opcache

# ตั้งค่า OPcache + JIT แยกไฟล์ .ini (ไม่เขียนทับ php.ini ตรง ๆ)
RUN { \
      echo 'zend_extension=opcache'; \
      echo 'opcache.enable=1'; \
      echo 'opcache.enable_cli=1'; \
      echo 'opcache.validate_timestamps=1'; \
      echo 'opcache.memory_consumption=256'; \
      echo 'opcache.interned_strings_buffer=16'; \
      echo 'opcache.max_accelerated_files=20000'; \
      echo 'opcache.jit_buffer_size=256M'; \
      echo 'opcache.jit=tracing'; \
    } > /usr/local/etc/php/conf.d/opcache.ini

WORKDIR /var/www/html
COPY ./.htaccess ./
COPY ./index.php ./
