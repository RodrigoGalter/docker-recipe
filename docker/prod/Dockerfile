FROM php:7.4.11-fpm

ARG NR_LICENSE_KEY
ARG NR_APPLICATION_NAME
ARG NR_VERSION=9.13.0.270

RUN apt-get update && \
    apt-get upgrade -y && \
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
    wget \
    libsodium-dev \
    libicu-dev \
    libpng-dev \
    libjpeg-dev \
    libjpeg62-turbo-dev \
    libzip-dev && \
    #pecl
    pecl install redis && \
    #Configure extensions
    docker-php-ext-configure intl && \
    docker-php-ext-configure gd --with-jpeg && \
    docker-php-ext-install pdo_mysql mysqli -j$(nproc) gd pcntl bcmath zip intl opcache && \
    #Cron
    echo "*/1 * * * * www-data /var/www/html/artisan schedule:run 2>&1" >> /etc/crontab && \
    #Composer
    curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer && \
    chmod +x /usr/local/bin/composer

#Supervisor
COPY supervisor /etc/supervisor/conf.d
COPY php/conf.d/custom.ini /usr/local/etc/php/conf.d/custom.ini

#NewRelic
RUN curl -L "https://download.newrelic.com/php_agent/archive/${NR_VERSION}/newrelic-php5-${NR_VERSION}-linux.tar.gz" | tar -C /tmp -zx && \
    NR_INSTALL_USE_CP_NOT_LN=1 NR_INSTALL_SILENT=1 /tmp/newrelic-php5-*/newrelic-install install && \
    rm -rf /tmp/newrelic-php5-* /tmp/nrinstall* && \
    sed -i \
        -e "s/newrelic.license =.*/newrelic.license = ${NR_LICENSE_KEY}/" \
        -e "s/newrelic.appname =.*/newrelic.appname = \"${NR_APPLICATION_NAME}\"/" \
        /usr/local/etc/php/conf.d/newrelic.ini

RUN apt-get upgrade -y && \
    apt-get autoclean && \
    apt-get autoremove -y && \
    apt-get purge -y \
        libsodium-dev \
        libicu-dev \
        libpng-dev \
        libjpeg-dev \
        libjpeg62-turbo-dev \
        libzip-dev && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /var/www/html

CMD /usr/sbin/cron && /usr/bin/supervisord && php-fpm