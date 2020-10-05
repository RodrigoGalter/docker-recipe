FROM php:7.4.11-fpm

RUN apt-get update && \
    apt-get install -y \
    locales \
    locales-all \
    supervisor -y \
    cron -y \
    vim \
    mcrypt \
    ssh \
    openssl \
    zip \
    libsodium-dev \
    libicu-dev \
    libpng-dev \
    libjpeg-dev \
    libjpeg62-turbo-dev \
    libzip-dev && \
    #pecl
    pecl install redis && \
    pecl install xdebug && \
    #Configure extensions
    docker-php-ext-configure intl && \
    docker-php-ext-configure gd --with-jpeg && \
    docker-php-ext-install pdo_mysql mysqli -j$(nproc) gd pcntl bcmath zip intl && \
    docker-php-ext-enable xdebug && \
    #Cron
    echo "*/1 * * * * www-data /var/www/html/artisan schedule:run 2>&1" >> /etc/crontab && \
    #Composer
    curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer && \
    chmod +x /usr/local/bin/composer

# Supervisor
COPY supervisor /etc/supervisor/conf.d
COPY php/conf.d/custom.ini /usr/local/etc/php/conf.d/custom.ini

RUN apt-get install -y wget && \
    wget https://phar.phpunit.de/phpunit-9.3.8.phar && \
    mv phpunit-9.3.8.phar /usr/local/bin/phpunit && \
    chmod +x /usr/local/bin/phpunit

#RUN cd /tmp && git clone https://github.com/swoole/swoole-src.git && \
#    cd swoole-src && \
#    git checkout v4.5.2 && \
#    phpize  && \
#    ./configure  --enable-openssl && \
#    make && make install && \
#    touch /usr/local/etc/php/conf.d/swoole.ini && \
#    echo 'extension=swoole.so' > /usr/local/etc/php/conf.d/swoole.ini

#RUN pecl install swoole

RUN apt-get upgrade -y && \
    apt-get autoclean && \
    apt-get autoremove -y && \
    apt-get purge -y \
        libsodium-dev \
        libicu-dev \
        libpng-dev \
        libjpeg-dev \
        libjpeg62-turbo-dev \
        libzip-dev \
        wget && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /var/www/html

CMD /usr/sbin/cron && /usr/bin/supervisord && php-fpm
